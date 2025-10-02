So I am confused. Fuck that here are the component.ts files and the service file and the model files. Clearly tell me what to do and where. 
schedule-oath.state.ts file:
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';
import { NotaryProfileDto } from '../../model/schedule-oath/schedule-oath.models';

export interface ScheduleOathState {
    applicantId: number | null;
    email: string | null;
    profile: NotaryProfileDto | null;

    visitTypeId: number | null;
    selectedSlot: any;
    holdEndTimeUtc: string | null;
    confirmation: any;
}

const initialState: ScheduleOathState = {
    applicantId: null,
    email: null,
    profile: null,

    visitTypeId: null,
    selectedSlot: null,
    holdEndTimeUtc: null,
    confirmation: null
};

const STORAGE_KEY = 'so_state';

function loadFromStorage(): ScheduleOathState {
    try {
        const raw = sessionStorage.getItem(STORAGE_KEY);
        if (!raw) return initialState;
        const parsed = JSON.parse(raw);
        return { ...initialState, ...parsed };
    } catch {
        return initialState;
    }
}

@Injectable({ providedIn: 'root' })
export class ScheduleOathStore {
    private subj = new BehaviorSubject<ScheduleOathState>(loadFromStorage());
    readonly state$ = this.subj.asObservable();

    get snapshot(): ScheduleOathState {
        return this.subj.value;
    }

    patch(p: Partial<ScheduleOathState>): void {
        const next = { ...this.subj.value, ...p };
        this.subj.next(next);
        try { sessionStorage.setItem(STORAGE_KEY, JSON.stringify(next)); } catch { }
    }

    reset(): void {
        this.subj.next({ ...initialState });
        try { sessionStorage.removeItem(STORAGE_KEY); } catch { }
    }
}
schedule-oath-confirm.component.ts file:
import { Component, NgZone, OnDestroy, OnInit } from '@angular/core';
import { Router } from '@angular/router';
import { interval, Subject, takeUntil } from 'rxjs';
import { ScheduleOathService } from '../../services/schedule-oath/schedule-oath.service';
import { ScheduleOathStore } from '../schedule-oath/schedule-oath.state';

@Component({
    selector: 'app-schedule-oath-confirm',
    templateUrl: './schedule-oath-confirm.component.html',
    styleUrls: ['./schedule-oath-confirm.component.css']
})
export class ScheduleOathConfirmComponent implements OnInit, OnDestroy {
    disclaimerAccepted = false;
    holdExpiresAt!: Date;
    remaining = '';
    expired = false;

    private destroy$ = new Subject<void>();

    constructor(
        public store: ScheduleOathStore,
        private service: ScheduleOathService,
        public router: Router,
        private zone: NgZone
    ) { }

    get slot() { return this.store.snapshot.selectedSlot; }
    get startIso() { return this.slot?.startTimeUtc ?? null; }
    get endIso() { return this.slot?.endTimeUtc ?? null; }

    ngOnInit(): void {
        const s = this.store.snapshot;
        if (!s.selectedSlot || !s.holdEndTimeUtc) {
            this.router.navigate(['/schedule-oath/date-time']);
            return;
        }
        this.holdExpiresAt = new Date(s.holdEndTimeUtc);

        this.zone.runOutsideAngular(() => {
            interval(1000).pipe(takeUntil(this.destroy$)).subscribe(() => {
                const diff = this.holdExpiresAt.getTime() - Date.now();
                this.expired = diff <= 0;
                this.remaining = this.formatDiff(Math.max(diff, 0));
            });
        });
    }

    private formatDiff(ms: number): string {
        const total = Math.floor(ms / 1000);
        const m = Math.floor(total / 60).toString().padStart(2, '0');
        const s = (total % 60).toString().padStart(2, '0');
        return `${m}:${s}`;
    }

    goBackToSlots() { this.router.navigate(['/schedule-oath/date-time']); }

    confirm() {
        const s = this.store.snapshot;
        if (this.expired) { this.goBackToSlots(); return; }

        this.service.confirmReservation({
            applicantId: s.applicantId!,
            reservationSlotId: s.selectedSlot!.reservationSlotId,
            visitTypeId: s.visitTypeId!,
            disclaimerAccepted: this.disclaimerAccepted
        }).subscribe({
            next: conf => {
                this.store.patch({ confirmation: conf });
                this.router.navigate(['/schedule-oath/confirmation']);
            },
            error: (err) => {
                if (err?.status === 409) this.goBackToSlots();
                else alert('Failed to confirm reservation.');
            }
        });
    }

    ngOnDestroy(): void { this.destroy$.next(); this.destroy$.complete(); }
}
schedule-oath-confirmation.component.ts file: 
import { Component } from '@angular/core';
import { Router } from '@angular/router';
import { ScheduleOathStore } from '../schedule-oath/schedule-oath.state';

@Component({
    selector: 'app-schedule-oath-confirmation',
    templateUrl: './schedule-oath-confirmation.component.html',
    styleUrls: ['./schedule-oath-confirmation.component.css']
})
export class ScheduleOathConfirmationComponent {
    constructor(public store: ScheduleOathStore, public router: Router) { }

    print() { window.print(); }
    done() {
        this.store.reset();  
        this.router.navigate(['/']);
    }
    startOver() { this.router.navigate(['/schedule-oath']); }
}
schedule-oath-date-time.component.ts file:
import {
    Component,
    OnDestroy,
    OnInit,
    ChangeDetectionStrategy,
    ChangeDetectorRef
} from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import { Subject, takeUntil } from 'rxjs';
import { ScheduleOathService } from '../../services/schedule-oath/schedule-oath.service';
import {
    OfficeDto,
    ReservationSlotDto,
    HoldResponseDto,
    ReservationConfirmationDto
} from '../../model/schedule-oath/schedule-oath.models';
import { ScheduleOathStore } from '../schedule-oath/schedule-oath.state';

type Dir = 'current' | 'next' | 'prev';

export type SchedulerItem = {
    id: number;
    slotId: number;
    title: string;
    start: Date;
    end: Date;
    isAllDay: boolean;
};

function isValidDate(d: Date): boolean {
    return d instanceof Date && !Number.isNaN(d.getTime());
}

@Component({
    selector: 'app-schedule-oath-date-time',
    templateUrl: './schedule-oath-date-time.component.html',
    styleUrls: ['./schedule-oath-date-time.component.css'],
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class ScheduleOathDateTimeComponent implements OnInit, OnDestroy {
    loading = true;
    errorMsg = '';

    offices: OfficeDto[] = [];
    selectedOfficeId: number | null = null;

    /** Events bound to the Scheduler via [kendoSchedulerBinding] */
    events: SchedulerItem[] = [];
    selectedDate: Date = new Date();

    weekLabel = '';
    hold: HoldResponseDto | null = null;
    acceptedDisclaimer = false;
    confirming = false;

    private applicantId = 0;
    private email = '';
    private visitTypeId = 0;
    private destroy$ = new Subject<void>();

    private static readonly MAX_EVENTS = 1000;

    constructor(
        private route: ActivatedRoute,
        private router: Router,
        private svc: ScheduleOathService,
        public store: ScheduleOathStore,
        private cdr: ChangeDetectorRef
    ) { }

    ngOnInit(): void {
        const qp = this.route.snapshot.queryParamMap;
        this.applicantId =
            Number(qp.get('applicantId')) || this.store.snapshot.applicantId || 0;
        this.email = qp.get('email') || this.store.snapshot.email || '';
        this.visitTypeId =
            Number(qp.get('visitTypeId')) || this.store.snapshot.visitTypeId || 0;

        if (!this.applicantId || !this.email || !this.visitTypeId) {
            this.loading = false;
            this.errorMsg =
                'Missing required data (applicantId / email / visitTypeId).';
            this.cdr.markForCheck();
            return;
        }

        // Persist for refreshes
        this.store.patch({
            applicantId: this.applicantId,
            email: this.email,
            visitTypeId: this.visitTypeId
        });

        this.loadOffices(() => this.loadSlotsWithFallback());
    }

    // ---------- data loads ----------
    private loadOffices(onDone?: () => void) {
        this.svc
            .getOffices()
            .pipe(takeUntil(this.destroy$))
            .subscribe({
                next: (offices) => {
                    this.offices = (offices || []).filter(
                        (o) => (o as any).isActive ?? true
                    );
                    if (!this.selectedOfficeId && this.offices.length) {
                        this.selectedOfficeId = this.offices[0].officeId;
                    }
                },
                error: () => { },
                complete: () => {
                    if (onDone) onDone();
                    this.cdr.markForCheck();
                }
            });
    }

    loadSlotsWithFallback(): void {
        this.loading = true;
        this.errorMsg = '';
        const weekOf = this.mondayIso(new Date());

        // current week
        this.searchWeek(weekOf, 'current', () => {
            if (this.events.length) {
                this.loading = false;
                this.cdr.markForCheck();
                return;
            }
            // next week
            this.searchWeek(weekOf, 'next', () => {
                this.loading = false;
                if (!this.events.length) {
                    this.errorMsg =
                        'No available slots for this or next week. Try a different week or office.';
                }
                this.cdr.markForCheck();
            });
        });
    }

    private searchWeek(weekOf: string, dir: Dir, done?: () => void): void {
        const req: any = {
            applicantId: this.applicantId,
            visitTypeId: this.visitTypeId,
            weekOf,
            // Keep your lowercase pageDirection since your sample shows that payload
            pageDirection: dir
        };
        if (this.selectedOfficeId) req.officeId = this.selectedOfficeId;

        this.svc
            .searchWeeklySlots(req)
            .pipe(takeUntil(this.destroy$))
            .subscribe({
                next: (resp) => {
                    // Type-safe: your SlotSearchResponse exposes weekStartUtc / weekEndUtc / slots
                    const startIso: string | undefined = resp?.weekStartUtc;
                    const endIso: string | undefined = resp?.weekEndUtc;

                    const wkStart = startIso ? new Date(startIso) : null;
                    const wkEnd = endIso ? new Date(endIso) : null;

                    if (!wkStart || !isValidDate(wkStart) || !wkEnd || !isValidDate(wkEnd)) {
                        this.events = [];
                        this.errorMsg = 'Server returned invalid week dates.';
                        return;
                    }

                    this.selectedDate = wkStart;
                    this.weekLabel = this.formatWeekLabel(startIso!, endIso!);

                    const rawSlots: any[] = Array.isArray(resp?.slots) ? resp.slots : [];

                    const items: SchedulerItem[] = rawSlots
                        .map((s: any) => {
                            const startUTC =
                                s.startTimeUtc ??
                                (s.scheduleDate && s.startTime
                                    ? `${s.scheduleDate}T${s.startTime}Z`
                                    : null) ??
                                s.start;
                            const endUTC =
                                s.endTimeUtc ??
                                (s.scheduleDate && s.endTime
                                    ? `${s.scheduleDate}T${s.endTime}Z`
                                    : null) ??
                                s.end;

                            if (!startUTC || !endUTC) return null;

                            const start = new Date(startUTC);
                            const end = new Date(endUTC);
                            if (!isValidDate(start) || !isValidDate(end)) return null;

                            const id = Number(s.reservationSlotId ?? s.slotId ?? s.id ?? 0);

                            return {
                                id,
                                slotId: id,
                                title: s.title ?? 'Available',
                                start,
                                end,
                                isAllDay: false
                            };
                        })
                        .filter((x): x is SchedulerItem => !!x)
                        .sort((a, b) => a.start.getTime() - b.start.getTime())
                        .slice(0, ScheduleOathDateTimeComponent.MAX_EVENTS);

                    this.events = items;
                    this.errorMsg = this.events.length
                        ? ''
                        : 'No available slots for this week. Try another week or office.';
                },
                error: (err) => {
                    this.errorMsg = err?.message || 'Failed to load slots.';
                    if (done) done();
                    this.cdr.markForCheck();
                },
                complete: () => {
                    if (done) done();
                    this.cdr.markForCheck();
                }
            });
    }

    // ---------- interactions ----------
    onOfficeChange(): void {
        this.loadSlotsWithFallback();
    }

    goWeek(dir: Dir): void {
        this.loading = true;
        const weekOf = this.mondayIso(new Date());
        this.searchWeek(weekOf, dir, () => {
            this.loading = false;
            this.cdr.markForCheck();
        });
    }

    onEventClick(e: any): void {
        // Only used if/when you add (eventClick) back
        const data = e?.event?.dataItem as SchedulerItem | undefined;
        if (!data) return;
        this.holdSlotById(data.slotId);
    }

    holdSlotById(slotId: number): void {
        if (!this.applicantId) return;
        this.hold = null;
        this.acceptedDisclaimer = false;

        this.svc
            .setHold(slotId, this.applicantId)
            .pipe(takeUntil(this.destroy$))
            .subscribe({
                next: (h) => {
                    this.hold = h;
                    this.cdr.markForCheck();
                },
                error: (err) => {
                    this.errorMsg = err?.message || 'Unable to hold slot.';
                    this.cdr.markForCheck();
                }
            });
    }

    releaseHold(): void {
        if (!this.hold) return;
        this.svc
            .releaseHold((this.hold as any).appointmentId)
            .pipe(takeUntil(this.destroy$))
            .subscribe({
                next: () => {
                    this.hold = null;
                    this.cdr.markForCheck();
                },
                error: () => {
                    this.hold = null;
                    this.cdr.markForCheck();
                }
            });
    }

    confirm(): void {
        if (!this.hold || !this.acceptedDisclaimer || this.confirming) return;
        this.confirming = true;
        this.cdr.markForCheck();

        this.svc
            .confirmReservation({
                appointmentId: (this.hold as any).appointmentId,
                disclaimerAccepted: true
            } as any)
            .pipe(takeUntil(this.destroy$))
            .subscribe({
                next: (conf: ReservationConfirmationDto) => {
                    this.router.navigate(['/schedule-oath/confirmation'], {
                        queryParams: {
                            reservationId: (conf as any).reservationId,
                            confirmationNumber: (conf as any).confirmationNumber
                        }
                    });
                },
                error: (err) => {
                    this.errorMsg = err?.message || 'Unable to confirm reservation.';
                    this.confirming = false;
                    this.cdr.markForCheck();
                },
                complete: () => {
                    this.confirming = false;
                    this.cdr.markForCheck();
                }
            });
    }

    // ---------- helpers ----------
    officeLabel(o: OfficeDto): string {
        const a = o as any;
        return a.value ?? a.name ?? a.description ?? `Office ${o.officeId}`;
    }
    trackOffice(_i: number, o: OfficeDto) {
        return o?.officeId;
    }

    private mondayIso(d: Date): string {
        const utc = new Date(
            Date.UTC(d.getUTCFullYear(), d.getUTCMonth(), d.getUTCDate())
        );
        const dow = utc.getUTCDay() || 7;
        if (dow !== 1) utc.setUTCDate(utc.getUTCDate() - (dow - 1));
        const y = utc.getUTCFullYear(),
            m = String(utc.getUTCMonth() + 1).padStart(2, '0'),
            day = String(utc.getUTCDate()).padStart(2, '0');
        return `${y}-${m}-${day}`;
    }
    private formatWeekLabel(startIso: string, endIso: string): string {
        try {
            const s = new Date(startIso),
                e = new Date(endIso);
            const fmt = (d: Date) =>
                d.toLocaleDateString(undefined, { month: 'short', day: 'numeric' });
            return `${fmt(s)}-${fmt(e)}, ${e.getFullYear()}`;
        } catch {
            return '';
        }
    }

    ngOnDestroy(): void {
        this.destroy$.next();
        this.destroy$.complete();
    }
}
schedule-oath-personal-info.component.ts file:
import { Component, OnDestroy, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators, FormControl } from '@angular/forms';
import { Router } from '@angular/router';
import { Subject, takeUntil } from 'rxjs';
import { timeout, finalize } from 'rxjs/operators';

import { ScheduleOathService } from '../../services/schedule-oath/schedule-oath.service';
import { ScheduleOathStore } from '../schedule-oath/schedule-oath.state';
import { NotaryProfileDto, VisitTypeDto } from '../../model/schedule-oath/schedule-oath.models';

type VisitCode = 'InPerson' | 'External' | 'Virtual';
type VisitVM = { id: number; code: VisitCode; label: string; help: string; raw?: VisitTypeDto };

@Component({
    selector: 'app-schedule-oath-personal-info',
    templateUrl: './schedule-oath-personal-info.component.html',
    styleUrls: ['./schedule-oath-personal-info.component.css']
})
export class ScheduleOathPersonalInfoComponent implements OnInit, OnDestroy {
    form!: FormGroup;
    submitted = false;

    profile: NotaryProfileDto | null = null;
    visitOptions: VisitVM[] = [];
    loading = true;

    private destroy$ = new Subject<void>();

    constructor(
        private fb: FormBuilder,
        private service: ScheduleOathService,
        public store: ScheduleOathStore,
        private router: Router
    ) { }

    ngOnInit(): void {
        const qp = new URLSearchParams(window.location.search);
        const applicantId = Number(qp.get('applicantId')) || this.store.snapshot.applicantId || 0;

        const emailFromUrl = (qp.get('email') || '').trim();
        const emailFromSession = (sessionStorage.getItem('Email') || '').trim();
        const email = emailFromUrl || emailFromSession || this.store.snapshot.email || '';

        // If URL missing email but available in session, rewrite once (supports deep links)
        if (applicantId && !emailFromUrl && emailFromSession) {
            this.router.navigate([], {
                queryParams: { email: emailFromSession },
                queryParamsHandling: 'merge',
                replaceUrl: true
            });
            return;
        }

        if (!applicantId) { this.loading = false; alert('Missing applicant id.'); return; }
        if (!email) { this.loading = false; alert('Missing email address. Please re-login or open from Profile Homepage.'); return; }

        // No disabled-on-load — control is enabled from the start
        this.form = this.fb.group({
            visitTypeId: new FormControl(
                this.store.snapshot.visitTypeId != null ? String(this.store.snapshot.visitTypeId) : '',
                Validators.required
            )
        });

        // Show defaults immediately
        this.visitOptions = this.defaultVisitOptions();

        // Persist identity early
        this.store.patch({ applicantId, email });

        // Load profile + visit types; just flip loading off in finalize()
        this.service.start(applicantId, email)
            .pipe(
                takeUntil(this.destroy$),
                timeout(5000),
                finalize(() => {
                    // runs on success OR error — no enable/disable here
                    this.loading = false;
                })
            )
            .subscribe({
                next: (res) => {
                    this.profile = res.profile ?? null;

                    const normalized = this.normalizeVisitTypes(res.visitTypes ?? []);
                    if (normalized.length) this.visitOptions = normalized;

                    // Default selection if none persisted
                    if (!this.form.value.visitTypeId && this.visitOptions.length) {
                        this.form.patchValue({ visitTypeId: String(this.visitOptions[0].id) });
                    } else if (typeof this.form.value.visitTypeId === 'number') {
                        this.form.patchValue({ visitTypeId: String(this.form.value.visitTypeId) });
                    }

                    // Persist profile for downstream steps
                    this.store.patch({ profile: this.profile });
                },
                error: () => {
                    // Keep defaults; still let user proceed
                    if (!this.form.value.visitTypeId && this.visitOptions.length) {
                        this.form.patchValue({ visitTypeId: String(this.visitOptions[0].id) });
                    }
                }
            });
    }

    onSelectVisitType(ev: Event): void {
        const val = (ev.target as HTMLInputElement | null)?.value ?? '';
        this.form.get('visitTypeId')!.setValue(String(val));
    }

    next(): void {
        this.submitted = true;
        if (this.form.invalid) {
            this.form.markAllAsTouched();
            return;
        }

        const visitTypeId = Number(this.form.value.visitTypeId);
        const { applicantId, email } = this.store.snapshot;

        this.store.patch({
            visitTypeId,
            selectedSlot: null,
            holdEndTimeUtc: null,
            confirmation: null
        });

        this.router.navigate(['/schedule-oath/date-time'], {
            queryParams: { applicantId, email, visitTypeId }
        });
    }

    cancel(): void {
        this.router.navigate(['/profile-homepage']);
    }

    // ---------- Helpers ----------
    private normalizeVisitTypes(list: VisitTypeDto[]): VisitVM[] {
        const onlyActive = (list || []).filter((v: any) => (v?.isActive ?? v?.IsActive ?? true));
        const toId = (v: any) => Number(v?.visitTypeId ?? v?.VisitTypeId ?? v?.id ?? 0);
        const toCode = (v: any): VisitCode | null => {
            const c = String(v?.code ?? v?.Code ?? v?.value ?? '').toLowerCase();
            if (c === 'inperson') return 'InPerson';
            if (c === 'external') return 'External';
            if (c === 'virtual') return 'Virtual';
            return null;
        };

        const opts: VisitVM[] = [];
        for (const v of onlyActive) {
            const id = toId(v);
            const code = toCode(v);
            if (!id || !code) continue;
            opts.push({
                id,
                code,
                label: code === 'InPerson' ? 'In Person' : code,
                help:
                    code === 'InPerson' ? 'Visit a designated office to take the oath' :
                        code === 'External' ? 'Take the oath with an approved partner' :
                            'Complete the oath remotely via video',
                raw: v
            });
        }
        const order: VisitCode[] = ['InPerson', 'External', 'Virtual'];
        return opts.sort((a, b) => order.indexOf(a.code) - order.indexOf(b.code));
    }

    private defaultVisitOptions(): VisitVM[] {
        return [
            { id: 1, code: 'InPerson', label: 'In Person', help: 'Visit a designated office to take the oath' },
            { id: 2, code: 'External', label: 'External', help: 'Take the oath with an approved partner' },
            { id: 3, code: 'Virtual', label: 'Virtual', help: 'Complete the oath remotely via video' }
        ];
    }

    ngOnDestroy(): void {
        this.destroy$.next();
        this.destroy$.complete();
    }
}
schedule-oath.models.ts file:
import { Type } from '@angular/core';

export interface OfficeDto {
    officeId: number;
    name: string;
    description: string;
    isActive: boolean;
    value?: string;
}

export interface ReservationSlotDto {
    reservationSlotId: number;
    officeId: number;
    scheduleDate: string;
    startTime: string;
    endTime: string;
    startTimeUtc?: string;
    endTimeUtc?: string;
    start?: string;
    end?: string;
    title?: string;
    id?: number;
}

export interface HoldRequestDto {
    applicantId: number;
    reservationSlotId: number;
}

export interface HoldResponseDto {
    appointmentId: number;
    officeId: number;
    officeName: string;
    officeAddress: string;
    officePhone: string;
    scheduleDateUtc: string;
    startTimeUtc: string;
    endTimeUtc: string;
    disclaimer: string;
    disclaimerAccepted: boolean;
}

export interface ConfirmReservationDto {
    appointmentId: number;
    disclaimerAccepted: boolean;
}

export interface ReservationConfirmationDto {
    reservationId: number;
    confirmationNumber: string;
    applicantId: number;
    officeId: number;
    officeName: string;
    scheduleDateUtc: string;
    startTimeUtc: string;
    endTimeUtc: string;
    disclaimer: string;
    disclaimerAccepted: boolean;
}

export interface WeekSlotsResponse {
    weekStartUtc: string;
    weekEndUtc: string;
    slots: ReservationSlotDto[];
}

export interface NotaryProfileDto {
    notaryId: number;
    fullName: string;
    // Add other fields as needed based on your API response
}

export interface VisitTypeDto {
    visitTypeId: number;
    description: string;
    // Add other fields as needed
}

export interface SlotFilterDto {
    startDateUtc: string;
    endDateUtc: string;
    officeId?: number;
    timeOfDay?: string;
}
