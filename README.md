Goal: To integrate other options in the oath-scheduler app. 
Details: Up Until now.. we only integrated the In-person flow for the oath-scheduler. Now, we will need to add 2 more 
flows. Virtual and External. In this prompt.. we will only deal with Virtual flow. 
The overall approach is same for virtual flow.. but there are few changes. I will mention the changes step by step: 

In Step - 1: We will need to pass the currently selected radio button from the schedule-oath-personal-info to 
schedule-oath-date-time component. 
In Step - 2: When we make the call to the offices endpoint.. and the offices endpoint returns the data like this [
  {
    "officeId": 1,
    "code": null,
    "value": "Boston",
    "description": null,
    "isActive": true
  },
  {
    "officeId": 2,
    "code": null,
    "value": "Fall River",
    "description": null,
    "isActive": true
  },
  {
    "officeId": 3,
    "code": null,
    "value": "Springfield",
    "description": null,
    "isActive": true
  },
  {
    "officeId": 4,
    "code": null,
    "value": "Virtual",
    "description": null,
    "isActive": true
  }
] we should not be displaying the virtual option when the user selected radio button in person in step - 1.. similarly 
when the user selected the option virtual in step -1 we should not be displaying the springfield, boston etc slots. So 
at that point when the user selected cirtual in step - 1 the dropdown should have only one value and that value is 
defaulted to virtual and we also make an automatic call to the backend search-slots with that officeId.
in Step - 3 : Just make sure you are passing a flag form previous step to this step and to downstreams as the 
downstream pages only have text based toggles that I can do it myself. 
in Step - 4 : Again. Pass a simple toggle from page 3 to here. 

Ask any clarifying questions you have before you implement. 

step - 1 schedule-oath-personal-info.component.ts file: 
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
        console.log('Radio', ctrl?.value);
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
step - 2 schedule-oath-date-time.component.ts file: 
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
    HoldResponseDto,
    ReservationConfirmationDto
} from '../../model/schedule-oath/schedule-oath.models';
import { ScheduleOathStore } from '../schedule-oath/schedule-oath.state';
import { OathStepTrackerService } from '../../services/helper-services/oath-schedule-step-tracker/oath-step-tracker.service';

type Dir = 'current' | 'next' | 'prev';

type TileVM = {
    slotId: number;
    start: Date;
    timeLabel: string;
    officeId: number | null;
    officeName: string;
    raw: any;
};

type DayVM = {
    key: string;       // YYYY-MM-DD
    label: string;     // Monday
    subLabel: string;  // May 5
    tiles: TileVM[];
};

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
    selectedOfficeId: number | null = null; // initial office for API calls

    // NEW: filter state (UI only)
    officeFilterOptions: { text: string; value: number | null }[] = [];
    selectedOfficeIdFilter: number | null = null; // null = All locations

    timeOfDayOptions = [
        { text: 'All available', value: 'All' as const },
        { text: 'AM', value: 'AM' as const },
        { text: 'PM', value: 'PM' as const }
    ];
    timeOfDayFilter: 'All' | 'AM' | 'PM' = 'All';

    // View model for grid
    daysVm: DayVM[] = [];
    selectedSlotId: number | null = null;

    // server/flow state
    weekLabel = '';
    hold: HoldResponseDto | null = null;
    acceptedDisclaimer = false;
    confirming = false;
    circum = true;
    dialogVisible = false;
    private applicantId = 0;
    private email = '';
    private visitTypeId = 0;
    private destroy$ = new Subject<void>();

    // track the currently viewed week (use server response)
    private currentWeekStartIso: string | null = null;

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
        this.applicantId = Number(qp.get('applicantId')) || this.store.snapshot.applicantId || 0;
        this.email = qp.get('email') || this.store.snapshot.email || '';
        this.visitTypeId = Number(qp.get('visitTypeId')) || this.store.snapshot.visitTypeId || 0;

        if (!this.applicantId || !this.email || !this.visitTypeId) {
            this.loading = false;
            this.errorMsg = 'Missing required data (applicantId / email / visitTypeId).';
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

    // ---------------- data loads ----------------
    private loadOffices(onDone?: () => void) {
        this.svc.getOffices().pipe(takeUntil(this.destroy$)).subscribe({
            next: (offices) => {
                this.offices = (offices || []).filter(o => (o as any).isActive ?? true);

                // API location dropdown (All + office values)
                this.officeFilterOptions = [
                    { text: 'All locations', value: null },
                    ...this.offices.map(o => ({ text: (o as any).value ?? o.name ?? `Office ${o.officeId}`, value: o.officeId }))
                ];

                if (!this.selectedOfficeId && this.offices.length) {
                    this.selectedOfficeId = this.offices[0].officeId;
                }
            },
            error: () => { },
            complete: () => { if (onDone) onDone(); this.cdr.markForCheck(); }
        });
    }

    /** Make a local Date at local midnight for the same calendar day as an ISO string */
    private localDateFromIsoDay(iso: string): Date {
        const d = new Date(iso); // UTC date-time from server
        return new Date(d.getUTCFullYear(), d.getUTCMonth(), d.getUTCDate()); // local midnight
    }

    /** Format YYYY-MM-DD from a *local* Date */
    private ymdLocal(d: Date): string {
        const y = d.getFullYear();
        const m = String(d.getMonth() + 1).padStart(2, '0');
        const day = String(d.getDate()).padStart(2, '0');
        return `${y}-${m}-${day}`;
    }

    loadSlotsWithFallback(): void {
        this.loading = true;
        this.errorMsg = '';

        const weekOf = this.currentWeekStartIso
            ? new Date(this.currentWeekStartIso)
            : new Date(); // first load = "current" from today

        const weekOfIso = this.mondayIso(weekOf);

        // current week
        this.searchWeek(weekOfIso, 'current', () => {
            if (this.daysVm.some(d => d.tiles.length)) {
                this.loading = false;
                this.cdr.markForCheck();
                return;
            }
            // next week
            this.searchWeek(weekOfIso, 'next', () => {
                this.loading = false;
                this.cdr.markForCheck();
            });
        });
    }

    private searchWeek(weekOf: string, dir: Dir, done?: () => void): void {
        const req: any = {
            applicantId: this.applicantId,
            visitTypeId: this.visitTypeId,
            weekOf,
            pageDirection: dir
        };
        if (this.selectedOfficeId) req.officeId = this.selectedOfficeId;

        this.svc.searchWeeklySlots(req).pipe(takeUntil(this.destroy$)).subscribe({
            next: (resp: any) => {
                const startIso: string | undefined = resp?.weekStartUtc ?? resp?.weekStart ?? resp?.weekStartISO;
                const endIso: string | undefined = resp?.weekEndUtc ?? resp?.weekEnd ?? resp?.weekEndISO;

                if (!startIso || !endIso) {
                    this.daysVm = this.emptyWeek();
                    this.weekLabel = '';
                    return;
                }

                this.currentWeekStartIso = startIso;
                this.weekLabel = this.formatWeekLabel(startIso, endIso);

                const rawSlots: any[] = Array.isArray(resp?.slots) ? resp.slots : [];
                const tiles = this.toTiles(rawSlots);
                const filtered = this.applyFilters(tiles);
                this.daysVm = this.groupIntoDays(filtered, startIso);
            },
            error: (err) => {
                this.errorMsg = err?.message || 'Failed to load slots.';
            },
            complete: () => { if (done) done(); this.cdr.markForCheck(); }
        });
    }

    // ---------------- filters & transforms ----------------
    onOfficeFilterChange(val: number | null) {
        this.selectedOfficeIdFilter = val;
        this.refreshViewOnly();
    }

    onTimeOfDayChange(val: 'All' | 'AM' | 'PM') {
        this.timeOfDayFilter = val;
        this.refreshViewOnly();
    }

    private refreshViewOnly() {
        // Rebuild day VM from last known raw slots: we don’t store raw; instead reload for simplicity.
        // If you prefer, keep the last `tiles` array in a field and re-run `applyFilters` + `groupIntoDays`.
        const weekOfIso = this.currentWeekStartIso ? this.mondayIso(new Date(this.currentWeekStartIso)) : this.mondayIso(new Date());
        this.searchWeek(weekOfIso, 'current');
    }

    private toTiles(rawSlots: any[]): TileVM[] {
        const officeNameById = (id: number | null | undefined) => {
            if (!id) return '—';
            const o = this.offices.find(x => x.officeId === id);
            return (o as any)?.value ?? (o as any)?.name ?? `Office ${id}`;
        };

        const parseStartLocal = (s: any): Date | null => {
            if (s.startTimeUtc || s.startUtc) {            // if API sends UTC, trust it
                const d = new Date(s.startTimeUtc || s.startUtc);
                return Number.isNaN(d.getTime()) ? null : d;
            }
            if (s.scheduleDate && s.startTime) {           // legacy/local shape ⇒ *no Z*
                const d = new Date(`${s.scheduleDate}T${s.startTime}`);
                return Number.isNaN(d.getTime()) ? null : d;
            }
            return null;
        };

        return rawSlots
            .map(s => {
                const start = parseStartLocal(s);
                if (!start) return null;

                const slotId = Number(s.reservationSlotId ?? s.slotId ?? s.id ?? 0);
                const officeId = s.office?.officeId ?? s.officeId ?? null;
                const officeName = s.office?.value ?? officeNameById(officeId);

                return {
                    slotId,
                    start,
                    timeLabel: this.formatTime(start), // still display-friendly
                    officeId,
                    officeName,
                    raw: s
                } as TileVM;
            })
            .filter((x): x is TileVM => !!x)
            .sort((a, b) => {
                const t = a.start.getTime() - b.start.getTime();
                return t !== 0 ? t : a.officeName.localeCompare(b.officeName);
            });
    }

    private applyFilters(list: TileVM[]): TileVM[] {
        let out = list;

        if (this.selectedOfficeIdFilter != null) {
            out = out.filter(t => t.officeId === this.selectedOfficeIdFilter);
        }

        if (this.timeOfDayFilter === 'AM') {
            out = out.filter(t => t.start.getHours() < 12);
        } else if (this.timeOfDayFilter === 'PM') {
            out = out.filter(t => t.start.getHours() >= 12);
        }

        return out;
    }

    private groupIntoDays(tiles: TileVM[], weekStartIso: string): DayVM[] {
        // ensure Monday local
        const mondayLocal = this.localDateFromIsoDay(weekStartIso);
        // If server accidentally sends a Sunday start, bump to Monday
        if (mondayLocal.getDay() === 0) { // 0 = Sunday, 1 = Monday
            mondayLocal.setDate(mondayLocal.getDate() + 1);
        }

        const days: DayVM[] = Array.from({ length: 5 }).map((_, i) => {
            const d = new Date(mondayLocal);
            d.setDate(mondayLocal.getDate() + i); // local day math
            return {
                key: this.ymdLocal(d), // local YYYY-MM-DD
                label: d.toLocaleDateString(undefined, { weekday: 'long' }),
                subLabel: d.toLocaleDateString(undefined, { month: 'short', day: 'numeric' }),
                tiles: []
            };
        });

        // Group tiles by API scheduleDate (authoritative), fallback to local date of start
        const byKey = new Map<string, TileVM[]>();
        for (const t of tiles) {
            const k = (t.raw?.scheduleDate as string) || this.ymdLocal(t.start);
            if (!byKey.has(k)) byKey.set(k, []);
            byKey.get(k)!.push(t);
        }

        for (const d of days) {
            d.tiles = (byKey.get(d.key) || []).slice();
        }

        return days;
    }

    // ---------------- interactions ----------------
    onTileClick(t: TileVM): void {
        const scheduleDate = t.raw?.scheduleDate || this.ymdLocal(t.start);
        const startTime = t.raw?.startTime || this.hhmmss(t.start);
        const endTime = t.raw?.endTime || '';

        //  Console-log it in backend response shape
        const slotPayload = {
            reservationSlotId: t.raw?.reservationSlotId ?? t.slotId,
            reservationId: t.raw?.reservationId ?? 0,
            office: {
                officeId: t.officeId ?? 0,
                value: t.officeName
            },
            visitTypeId: t.raw?.visitTypeId ?? 0,
            scheduleDate,
            startTime,
            endTime
        };

        console.log('Slot clicked (API shape):', slotPayload);

        // existing selection/hold behavior below 
        if (this.selectedSlotId === t.slotId) {
            this.selectedSlotId = null;
            if (this.hold) this.releaseHold();
            return;
        }

        this.selectedSlotId = t.slotId;
        this.holdSlotById(t.slotId);
    }

    onOfficeChange(): void {
        // kept for compatibility (not used by new UI)
        this.loadSlotsWithFallback();
    }

    goPrevWeek(): void {
        const base = this.currentWeekStartIso
            ? new Date(this.currentWeekStartIso)
            : new Date();
        this.searchWeek(this.mondayIso(base), 'prev');
    }

    goNextWeek(): void {
        const base = this.currentWeekStartIso
            ? new Date(this.currentWeekStartIso)
            : new Date();
        this.searchWeek(this.mondayIso(base), 'next');
    }

    holdSlotById(slotId: number): void {
        if (!this.applicantId) return;
        this.hold = null;
        this.acceptedDisclaimer = false;

        this.svc.setHold(slotId, this.applicantId).pipe(takeUntil(this.destroy$)).subscribe({
            next: (h) => {
                this.openDialog();
                this.hold = h;
                // persist minimal selection in the store for confirm step
                const chosen = this.findTile(slotId);
                this.store.patch({
                    appointmentId: (h as any).appointmentId ?? null,
                    holdEndTimeUtc: (h as any).holdEndTimeUtc ?? null,
                    selectedSlot: chosen
                        ? {
                            reservationSlotId: chosen.slotId,
                            startTimeUtc: chosen.start.toISOString(),
                            endTimeUtc: null,
                            title: 'Available',
                            officeName: chosen.officeName
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
        this.svc.releaseHold((this.hold as any).appointmentId).pipe(takeUntil(this.destroy$)).subscribe({
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

        this.svc.confirmReservation({
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

    // ---------------- helpers ----------------
    private findTile(slotId: number): TileVM | null {
        for (const d of this.daysVm) {
            const t = d.tiles.find(x => x.slotId === slotId);
            if (t) return t;
        }
        return null;
    }

    private mondayIso(d: Date): string {
        const utc = new Date(Date.UTC(d.getUTCFullYear(), d.getUTCMonth(), d.getUTCDate()));
        const dow = utc.getUTCDay() || 7;
        if (dow !== 1) utc.setUTCDate(utc.getUTCDate() - (dow - 1));
        const y = utc.getUTCFullYear(),
            m = String(utc.getUTCMonth() + 1).padStart(2, '0'),
            day = String(utc.getUTCDate()).padStart(2, '0');
        return `${y}-${m}-${day}`;
    }

    private formatWeekLabel(startIso: string, endIso: string): string {
        try {
            const s = this.localDateFromIsoDay(startIso);
            const e = this.localDateFromIsoDay(endIso);
            const fmt = (d: Date) => d.toLocaleDateString(undefined, {
                month: 'long',
                day: 'numeric',
                year: 'numeric'
            });
            return `${fmt(s)} - ${fmt(e)}`;
        } catch {
            return '';
        }
    }

    private formatTime(d: Date): string {
        return d.toLocaleTimeString(undefined, { hour: 'numeric', minute: '2-digit' });
    }

    private emptyWeek(): DayVM[] {
        const start = new Date();
        const monday = new Date(this.mondayIso(start));
        return Array.from({ length: 5 }).map((_, i) => {
            const d = new Date(monday);
            d.setUTCDate(monday.getUTCDate() + i);
            return {
                key: d.toISOString().slice(0, 10),
                label: d.toLocaleDateString(undefined, { weekday: 'long' }),
                subLabel: d.toLocaleDateString(undefined, { month: 'short', day: 'numeric' }),
                tiles: []
            };
        });
    }

    goToConfirm(): void {
        // must have a hold to proceed
        if (!this.hold) { return; }

        // mark steps and navigate to the Confirm page
        this.stepTracker.completeDateTimeAndActivateConfirm();
        this.router.navigate(['/schedule-oath/confirm']);
    }

    private hhmmss(d: Date): string {
        const h = String(d.getHours()).padStart(2, '0');
        const m = String(d.getMinutes()).padStart(2, '0');
        return `${h}:${m}:00`;
    }

    onCancel(): void {
        this.openDialog();
    }

    openDialog(): void {
        this.dialogVisible = true;
    }

    closeDialog(): void {
        this.releaseHold();
        this.dialogVisible = false;
    }

    ngOnDestroy(): void {
        this.destroy$.next();
        this.destroy$.complete();
    }
}
step - 3 schedule-oath-confirm.component.ts file: 
import { Component, NgZone, OnDestroy, OnInit } from '@angular/core';
import { Router } from '@angular/router';
import { interval, startWith, Subject, takeUntil } from 'rxjs';
import { ScheduleOathService } from '../../services/schedule-oath/schedule-oath.service';
import { ScheduleOathStore } from '../schedule-oath/schedule-oath.state';
import { OathStepTrackerService } from '../../services/helper-services/oath-schedule-step-tracker/oath-step-tracker.service';

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
        private zone: NgZone,
        private stepTracker: OathStepTrackerService
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

            interval(1000).pipe(startWith(0), takeUntil(this.destroy$)).subscribe(() => {
                const diff = this.holdExpiresAt.getTime() - Date.now();
                this.expired = diff <= 0;
                this.remaining = this.formatDiff(Math.max(diff, 0));
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

        // Must have an appointmentId to confirm
        if (!s.appointmentId) { this.goBackToSlots(); return; }

        this.service.confirmReservation({
            appointmentId: s.appointmentId,
            disclaimerAccepted: this.disclaimerAccepted
        }).subscribe({
            next: conf => {
                this.store.patch({ confirmation: conf });
                this.stepTracker.completeConfirmAndActivateFinal();
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
step - 4 schedule-oath-confirmation.component.ts file
import { Component } from '@angular/core';
import { Router } from '@angular/router';
import { ScheduleOathStore } from '../schedule-oath/schedule-oath.state';

@Component({
    selector: 'app-schedule-oath-confirmation',
    templateUrl: './schedule-oath-confirmation.component.html',
    styleUrls: ['./schedule-oath-confirmation.component.css']
})


export class ScheduleOathConfirmationComponent {
    mockData = {
        "confirmationNumber": "12",
        "officeName": "Boston",
        "startTimeUtc": "2021-05-16T04:10:23.000Z",
        "endTimeUtc": "2021-05-16T05:10:23.000Z"
    }
    copied = false;
    constructor(public store: ScheduleOathStore, public router: Router) {

    }

    print() { window.print(); }
    done() {
        this.store.reset();  
        this.router.navigate(['/']);
    }

    async copyConf(text: string | number | null | undefined) {
        const value = (text ?? '').toString().trim();
        if (!value) return;

        try {
            if (navigator.clipboard && window.isSecureContext) {
                await navigator.clipboard.writeText(value);
            } else {
                const ta = document.createElement('textarea');
                ta.value = value;
                // keep off-screen but selectable
                ta.style.position = 'fixed';
                ta.style.left = '-9999px';
                ta.style.opacity = '0';
                document.body.appendChild(ta);
                ta.select();
                document.execCommand('copy');
                document.body.removeChild(ta);
            }
            this.showCopiedHint();
        } catch (err) {
            console.error('Copy failed:', err);
        }
    }

    private showCopiedHint() {
        this.copied = true;
        setTimeout(() => (this.copied = false), 1500);
    }
    startOver() { this.router.navigate(['/schedule-oath']); }

    takeToProfilePage() {
        this.router.navigate(['/profile-homepage']);
    }
}
