Goal: Look at the code here and tell me what the code is doing on click on the next button and what network call is being made ask me to 
paste the code for a certain service or a certain component or a certain model if you need it. The end result you should be able to explain
what is going here on click of next. 

Ask me any clarifying questions you have 

schedule-oath-personal-info.component.html file: 
<section class="so-personal-info">
    <!-- Non-blocking skeleton -->
    <div class="skeleton" *ngIf="loading" aria-hidden="true">Loading…</div>

    <form *ngIf="!loading" [formGroup]="form" (ngSubmit)="next()" novalidate>
        <div class="welcome">Welcome {{profile?.firstName}},</div>
        <div class="welcome post-salutation">
            Your oath is due latest by <u>{{profile?.oathDueDateUtc | date:'MMM dd, yyyy \'by\' hh:mm a' }}</u> local time.
            Please confirm your information below, select the mode of oath and proceed.
        </div>

        <div class="profile-information-section">
            <div class="profile-information-heading">Profile Information</div>
            <div class="profile-information-data">
                <div class="data-column">
                    <div class="data-row label-heading">Name</div>
                    <div class="data-row">{{profile?.firstName}}</div>
                </div>
                <div class="data-column">
                    <div class="data-row label-heading">Date of Birth</div>
                    <div class="data-row">{{dobDisplay}}</div>
                </div>
                <div class="data-column">
                    <div class="data-row label-heading">Email</div>
                    <div class="data-row">{{store.snapshot.email}}</div>
                </div>
                <div class="data-column">
                    <div class="data-row label-heading">Phone</div>
                    <div class="data-row">{{phone1Display}}</div>
                </div>
            </div>
        </div>

        <div class="visit-types">
            <div class="profile-information-heading">Mode of Oath <sup class="asterisk">*</sup></div>
            <div class="options">
                <div class="option" *ngFor="let opt of visitOptions">
                    <kendo-radiobutton formControlName="visitTypeId"
                                       [name]="'visitTypeId'"
                                       [value]="'' + opt.id"
                                       (click)="onRadioClick('' + opt.id, $event)"
                                       (blur)="onRadioBlur()">
                    </kendo-radiobutton>

                    <!-- Whole label remains clickable -->
                    <label class="text"
                           [class.selected]="form.value.visitTypeId === '' + opt.id"
                           (click)="onRadioClick('' + opt.id, $event)">
                        <div class="title">{{ opt.label }}</div>
                        <div class="help" *ngIf="opt.help">{{ opt.help }}</div>
                    </label>
                </div>
            </div>

            <div class="error" *ngIf="submitted && form.get('visitTypeId')?.invalid">
                Please select a visit type to continue.
            </div>
        </div>

        <div class="actions">
            <button type="button" kendoButton class="previous-button" (click)="cancel()">Previous</button>
            <button type="submit" kendoButton class="forward-button" [disabled]="form.invalid">Next</button>
        </div>
    </form>
</section>
schedule-oath-personal-info.component.ts file:
import { Component, OnDestroy, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators, FormControl } from '@angular/forms';
import { Router } from '@angular/router';
import { Subject, takeUntil } from 'rxjs';
import { timeout, finalize } from 'rxjs/operators';

import { ScheduleOathService } from '../../services/schedule-oath/schedule-oath.service';
import { ScheduleOathStore } from '../schedule-oath/schedule-oath.state';
import { NotaryProfileDto, VisitTypeDto } from '../../model/schedule-oath/schedule-oath.models';
import { OathStepTrackerService } from '../../services/helper-services/oath-schedule-step-tracker/oath-step-tracker.service';

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
    dobDisplay: string = '';
    phone1Display: string = '';

    private destroy$ = new Subject<void>();

    constructor(
        private fb: FormBuilder,
        private service: ScheduleOathService,
        public store: ScheduleOathStore,
        private router: Router,
        private stepTracker: OathStepTrackerService
    ) { }

    ngOnInit(): void {
        const qp = new URLSearchParams(window.location.search);
        const applicantId = Number(qp.get('applicantId')) || this.store.snapshot.applicantId || 0;

        const emailFromUrl = (qp.get('email') || '').trim();
        const emailFromSession = (sessionStorage.getItem('Email') || '').trim();
        const email = emailFromUrl || emailFromSession || this.store.snapshot.email || '';
        const firstName = this.store.snapshot.profile?.firstName;
        const isModifyMode = (qp.get('modify') || 'false').toLowerCase() === 'true';
        this.getDobandPhone();

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
                this.store.snapshot.visitTypeId != null && isModifyMode
                    ? String(this.store.snapshot.visitTypeId)  // only preselect in modify mode
                    : '',                                      // otherwise leave empty
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
                    const saved = this.store.snapshot.visitTypeId;
                    if (isModifyMode && saved && !this.form.value.visitTypeId) {
                        this.form.patchValue({ visitTypeId: String(saved) }, { emitEvent: false });
                    }

                    // Persist profile for downstream steps
                    this.store.patch({ profile: this.profile });
                },
                error: () => {
                    // Keep defaults; still let user proceed
                    alert('Issue in saving');
                }
            });
    }

    getDobandPhone(): void {
        const raw = sessionStorage.getItem('ProfileDetails');

        if (raw) {
            try {
                const profile = JSON.parse(raw);
                const personalInfo = profile?.userHomeProfileDto?.personalInfoDetails;

                this.dobDisplay = personalInfo?.dateOfBirthDisplay ?? null;
                this.phone1Display = personalInfo?.phone1Display ?? null;
            } catch (err) {
                console.error('Invalid ProfileDetails JSON', err);
            }
        } else {
            console.warn('No ProfileDetails found in sessionStorage');
        }
    }

    onRadioClick(id: string, event: Event): void {
        const ctrl = this.form.get('visitTypeId');
        if (!ctrl) return;

        const current = ctrl.value;
        if (current === id) {
            event.preventDefault();
            event.stopPropagation();

            setTimeout(() => {
                ctrl.setValue('', { emitEvent: true });
                ctrl.markAsPristine();
                ctrl.markAsUntouched();
                ctrl.updateValueAndValidity({ emitEvent: true });
            });
        } else {
            ctrl.setValue(id, { emitEvent: true });
        }
    }

    onRadioBlur(): void {
        const ctrl = this.form.get('visitTypeId');
        if (!ctrl) return;
        if (!ctrl.value) {
            ctrl.markAsPristine();
            ctrl.markAsUntouched();
        }
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

        this.stepTracker.completePersonalAndActivateDateTime();

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
                    code === 'InPerson' ? 'at a location closer to you' :
                        code === 'External' ? 'I will find my own commissioner to qualify' :
                            'with a virtual oath you maybe able to schedule an oath at any location',
                raw: v
            });
        }
        const order: VisitCode[] = ['InPerson', 'Virtual','External'];
        return opts.sort((a, b) => order.indexOf(a.code) - order.indexOf(b.code));
    }

    private defaultVisitOptions(): VisitVM[] {
        return [
            { id: 1, code: 'InPerson', label: 'In Person', help: 'at a location closer to you' },
            { id: 2, code: 'Virtual', label: 'Virtual', help: 'with a virtual oath you maybe able to schedule an oath at any location' },
            { id: 3, code: 'External', label: 'External', help: 'I will find my own commissioner to qualify' }
        ];
    }

    ngOnDestroy(): void {
        this.destroy$.next();
        this.destroy$.complete();
    }
}
schedule-oath.service.ts file:
import { Injectable } from '@angular/core';
import { HttpClient, HttpErrorResponse, HttpParams } from '@angular/common/http';
import { environment } from '../../../environments/environment';
import { Observable, throwError } from 'rxjs';
import { catchError, map } from 'rxjs/operators';

import {
    NotaryProfileDto,
    VisitTypeDto,
    OfficeDto,
    ReservationSlotDto,
    SlotFilterDto,          // UI filter (date range, etc.)
    HoldResponseDto,        // contains appointmentId, holdEndTimeUtc
    ConfirmReservationDto,  // contains appointmentId, disclaimerAccepted
    ReservationConfirmationDto, // confirm response
} from '../../model/schedule-oath/schedule-oath.models';

type PageDirection = 'current' | 'next' | 'prev';

interface StartResponse {
    profile: NotaryProfileDto;
    visitTypes: VisitTypeDto[];
}

interface SlotSearchRequest {
    applicantId: number;
    visitTypeId: number;
    weekOf: string;                // 'YYYY-MM-DD' (UTC date; server normalizes to Monday)
    pageDirection: PageDirection;  // 'current' | 'next' | 'prev'
    officeId?: number;
    timeOfDay?: 'AM' | 'PM' | 'Any' | 'morning' | 'afternoon' | 'evening' | 'all';
}

interface SlotSearchResponse {
    weekStartUtc: string;          // ISO
    weekEndUtc: string;            // ISO
    slots: ReservationSlotDto[];   // your UI shape
}

@Injectable({ providedIn: 'root' })
export class ScheduleOathService {
    private readonly base = `${environment.apiurl}/api/ExternalReservations`;

    constructor(private http: HttpClient) { }

    // -------------------------
    // Error handling
    // -------------------------
    private handleError(err: HttpErrorResponse): Observable<never> {
        const message =
            (err?.error && (err.error.title || err.error.message)) ||
            err?.statusText ||
            'Request failed';
        return throwError(() => ({ ...err, message }));
    }

    // -------------------------
    // Step 1: start/profile + visit types
    // GET /api/ExternalReservations/start?applicantId=&email=
    // -------------------------
    start(applicantId: number, email: string): Observable<StartResponse> {
        const params = new HttpParams()
            .set('applicantId', String(applicantId))
            .set('email', email);

        return this.http.get<any>(`${this.base}/start`, { params }).pipe(
            map(res => {
                const profile: NotaryProfileDto =
                    res?.profile ?? res?.notaryProfile ?? res?.data?.profile ?? null;
                const visitTypes: VisitTypeDto[] =
                    res?.visitTypes ?? res?.allowedVisitTypes ?? res?.data?.visitTypes ?? [];
                return { profile, visitTypes };
            }),
            catchError(err => this.handleError(err))
        );
    }

    /** Convenience: just the profile (calls start under the hood). */
    getNotaryProfileByIdAndEmail(applicantId: number, email: string): Observable<NotaryProfileDto> {
        return this.start(applicantId, email).pipe(
            map(r => r.profile),
            catchError(err => this.handleError(err))
        );
    }

    // -------------------------
    // Optional: lookups
    // GET /api/ExternalReservations/offices
    // -------------------------
    getOffices(): Observable<OfficeDto[]> {
        return this.http
            .get<OfficeDto[]>(`${this.base}/offices`)
            .pipe(catchError(err => this.handleError(err)));
    }

    // -------------------------
    // Step 2: search weekly slots
    // POST /api/ExternalReservations/search-slots
    // -------------------------
    searchWeeklySlots(req: SlotSearchRequest): Observable<SlotSearchResponse> {
        return this.http
            .post<SlotSearchResponse>(`${this.base}/search-slots`, req)
            .pipe(catchError(err => this.handleError(err)));
    }

    /**
     * Back-compat wrapper for UIs that still pass a date range.
     * We translate the range into a week-of (Monday) + 'current' page,
     * call weekly search, then (optionally) filter by start/end locally.
     */
    getAvailableSlots(filter: Partial<SlotFilterDto> & Record<string, any>): Observable<ReservationSlotDto[]> {
        const applicantId = Number(this.pick(filter, 'applicantId', 'ApplicantId') ?? 0);
        const visitTypeId = Number(this.pick(filter, 'visitTypeId', 'VisitTypeId') ?? 0);
        const officeId = this.numOrUndef(this.pick(filter, 'officeId', 'OfficeId'));
        const timeOfDay = (this.pick(filter, 'timeOfDay', 'TimeOfDay') ?? '').toString().trim() || undefined;

        const startRaw = this.pick(filter, 'startDateUtc', 'StartDateUtc', 'startDate', 'fromDate');
        const endRaw = this.pick(filter, 'endDateUtc', 'EndDateUtc', 'endDate', 'toDate');
        const start = startRaw ? new Date(startRaw) : new Date();
        const weekOf = this.mondayIso(start);

        const req: SlotSearchRequest = {
            applicantId,
            visitTypeId,
            weekOf,
            pageDirection: 'current',
            ...(officeId ? { officeId } : {}),
            ...(timeOfDay ? { timeOfDay } : {})
        };

        return this.searchWeeklySlots(req).pipe(
            map(resp => {
                if (!startRaw || !endRaw) return resp.slots;

                const from = new Date(startRaw).getTime();
                const to = new Date(endRaw).getTime();

                // ✅ Use model's UTC fields; fallback if backend returns legacy shapes.
                return resp.slots.filter((s) => {
                    const [sStartMs, sEndMs] = this.slotRangeMillis(s);
                    // if we couldn't parse, keep the slot (be permissive) or exclude – choose policy:
                    if (!Number.isFinite(sStartMs) || !Number.isFinite(sEndMs)) return false;
                    return sEndMs > from && sStartMs < to; // overlap
                });
            }),
            catchError(err => this.handleError(err))
        );
    }

    // -------------------------
    // Step 3: holds
    // POST /api/ExternalReservations/holds
    // DELETE /api/ExternalReservations/holds/{holdId}
    // -------------------------
    setHold(reservationSlotId: number, applicantId: number): Observable<HoldResponseDto> {
        const body = { applicantId, reservationSlotId };
        return this.http
            .post<HoldResponseDto>(`${this.base}/holds`, body)
            .pipe(catchError(err => this.handleError(err)));
    }

    releaseHold(holdId: number): Observable<void> {
        return this.http
            .delete<void>(`${this.base}/holds/${holdId}`)
            .pipe(catchError(err => this.handleError(err)));
    }

    // -------------------------
    // Step 4: confirm
    // POST /api/ExternalReservations/confirm?appointmentId=&disclaimerAccepted=
    // -------------------------
    confirmReservation(payload: ConfirmReservationDto): Observable<ReservationConfirmationDto> {
        const params = new HttpParams()
            .set('appointmentId', String((payload as any).appointmentId))
            .set('disclaimerAccepted', String((payload as any).disclaimerAccepted ?? true));

        return this.http
            .post<ReservationConfirmationDto>(`${this.base}/confirm`, null, { params })
            .pipe(catchError(err => this.handleError(err)));
    }

    // -------------------------
    // Optional: reservations (history/upcoming)
    // GET /api/ExternalReservations/applicants/{applicantId}/reservations
    // DELETE /api/ExternalReservations/reservations/{reservationId}?reason=
    // -------------------------
    getReservations(applicantId: number) {
        return this.http
            .get<any[]>(`${this.base}/applicants/${applicantId}/reservations`)
            .pipe(catchError(err => this.handleError(err)));
    }

    cancelReservation(reservationId: number, reason?: string) {
        const params = reason ? new HttpParams().set('reason', reason) : undefined;
        return this.http
            .delete<void>(`${this.base}/reservations/${reservationId}`, { params })
            .pipe(catchError(err => this.handleError(err)));
    }

    // -------------------------
    // helpers
    // -------------------------
    private pick(obj: Record<string, any>, ...keys: string[]) {
        for (const k of keys) {
            const v = obj?.[k];
            if (v !== undefined && v !== null && v !== '') return v;
        }
        return undefined;
    }

    private numOrUndef(v: any): number | undefined {
        const n = Number(v);
        return Number.isFinite(n) && n > 0 ? n : undefined;
    }

    /** Returns 'YYYY-MM-DD' (UTC) for the Monday of the week containing `d`. */
    private mondayIso(d: Date): string {
        const utc = new Date(Date.UTC(d.getUTCFullYear(), d.getUTCMonth(), d.getUTCDate()));
        const dow = utc.getUTCDay() || 7; // Sun=0 -> 7
        if (dow !== 1) utc.setUTCDate(utc.getUTCDate() - (dow - 1));
        const y = utc.getUTCFullYear();
        const m = String(utc.getUTCMonth() + 1).padStart(2, '0');
        const day = String(utc.getUTCDate()).padStart(2, '0');
        return `${y}-${m}-${day}`;
    }

    /**
     * Parse a slot's start/end in milliseconds, supporting both:
     * - current model fields: startTimeUtc / endTimeUtc (ISO)
     * - legacy fields: scheduleDate + startTime / endTime (HH:mm:ss)
     */
    private slotRangeMillis(slot: ReservationSlotDto): [number, number] {
        const s: any = slot as any;

        const startIso =
            s.startTimeUtc || s.startUtc ||
            (s.scheduleDate && s.startTime ? `${s.scheduleDate}T${s.startTime}Z` : undefined);

        const endIso =
            s.endTimeUtc || s.endUtc ||
            (s.scheduleDate && s.endTime ? `${s.scheduleDate}T${s.endTime}Z` : undefined);

        const startMs = startIso ? Date.parse(startIso) : Number.NaN;
        const endMs = endIso ? Date.parse(endIso) : Number.NaN;

        return [startMs, endMs];
    }
}
schedule-oath.state.ts file:
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';
import { NotaryProfileDto } from '../../model/schedule-oath/schedule-oath.models';

export interface ScheduleOathState {
    applicantId: number | null;
    email: string | null;
    profile: NotaryProfileDto | null;
    appointmentId: number | null;
    visitTypeId: number | null;
    selectedSlot: any;
    holdEndTimeUtc: string | null;
    confirmation: any;
}

const initialState: ScheduleOathState = {
    applicantId: null,
    email: null,
    profile: null,
    appointmentId: null,
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
step - 2 schedule-oath-date-time.component.html file: 
<div class="wrapper" *ngIf="!loading; else busy">
    <div class="header">
        <h2>Pick a Date & Time</h2>
        <div class="week-label" *ngIf="weekLabel">{{ weekLabel }}</div>

        <div class="controls">
            <label>
                Office:
                <select class="k-input"
                        [ngModel]="selectedOfficeId"
                        (ngModelChange)="selectedOfficeId = $event; onOfficeChange()">
                    <option *ngFor="let o of offices; trackBy: trackOffice" [value]="o.officeId">
                        {{ officeLabel(o) }}
                    </option>
                </select>
            </label>

            <div class="spacer"></div>

            <button kendoButton look="flat" (click)="goWeek('prev')" aria-label="Previous week">previous</button>
            <button kendoButton look="flat" (click)="goWeek('current')" aria-label="This week">This Week</button>
            <button kendoButton look="flat" (click)="goWeek('next')" aria-label="Next week">next</button>
        </div>
    </div>

    <div class="content">
        <!-- Add the eventClick binding to allow users to select an appointment slot -->
        <kendo-scheduler [kendoSchedulerBinding]="events"
                         [selectedDate]="selectedDate"
                         (eventClick)="onEventClick($event)"
                         style="height: 650px">
            <kendo-scheduler-week-view [slotDuration]="30"></kendo-scheduler-week-view>
        </kendo-scheduler>

        <div class="error" *ngIf="errorMsg">{{ errorMsg }}</div>
    </div>

    <div class="hold" *ngIf="hold as h">
        <h3>Selected Slot</h3>
        <p>Appointment ID: <strong>{{ h.appointmentId }}</strong></p>

        <label class="disclaimer">
            <input type="checkbox" [(ngModel)]="acceptedDisclaimer" />
            I understand this hold will be released if I don't confirm promptly.
        </label>

        <div class="hold-actions">
            <button kendoButton (click)="releaseHold()">Release Hold</button>
            <button kendoButton
                    themeColor="primary"
                    [disabled]="!acceptedDisclaimer || confirming"
                    (click)="confirm()">
                Confirm Reservation
            </button>
        </div>
    </div>
</div>

<ng-template #busy>
    <div class="busy">
        <kendo-loader type="infinite-spinner"></kendo-loader>
        <div>Loading available offices and slots...</div>
    </div>
</ng-template>
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
import { OathStepTrackerService } from '../../services/helper-services/oath-schedule-step-tracker/oath-step-tracker.service';

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
        private cdr: ChangeDetectorRef,
        private stepTracker: OathStepTrackerService
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

                    // Find the selected slot details from current events (for display on confirm page)
                    const chosen = this.events.find(ev => ev.slotId === slotId) || null;

                    // Persist for the confirm step/page
                    this.store.patch({
                        appointmentId: (h as any).appointmentId ?? null,
                        holdEndTimeUtc: (h as any).holdEndTimeUtc ?? null, // backend may or may not return this
                        selectedSlot: chosen
                            ? {
                                reservationSlotId: chosen.slotId,
                                startTimeUtc: chosen.start?.toISOString?.() ?? null,
                                endTimeUtc: chosen.end?.toISOString?.() ?? null,
                                title: chosen.title ?? 'Available'
                            }
                            : null
                    });

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
                    this.stepTracker.completeDateTimeAndActivateConfirm();
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
