Issue: When the page slots load and I click on a slot.. the hidden section below the grid appears. I then click on the disclaimer 
and click confirm reservation.. it should take me to the schedule-oath-confirm page. But instead it is taking me to the paersonal 
information page with the applicantId and email in the query params and says loading.. there are no errors in the UI console as well. 

Fix this. 

schedule-oath-date-time.component.html file:
<div class="wrapper" *ngIf="!loading; else busy">
    <h2 class="date-time-heading">Select a Date and Time</h2>
    <div class="header">
        <!-- Filters -->
        <div class="filters">
            <kendo-dropdownlist [data]="officeFilterOptions"
                                textField="text"
                                valueField="value"
                                [value]="selectedOfficeIdFilter"
                                [valuePrimitive]="true"
                                (valueChange)="onOfficeFilterChange($event)">
            </kendo-dropdownlist>

            <kendo-dropdownlist [data]="timeOfDayOptions"
                                textField="text"
                                valueField="value"
                                [value]="timeOfDayFilter"
                                [valuePrimitive]="true"
                                (valueChange)="onTimeOfDayChange($event)">
            </kendo-dropdownlist>
            <div class="week-label" *ngIf="weekLabel">{{ weekLabel }}</div>
        </div>
    </div>

    <div class="board-row">
        <button kendoButton fillMode="flat" class="nav-btn prev" (click)="goPrevWeek()">&larr; Previous week</button>

        <div class="grid-board">
            <!-- 5 fixed columns (Mon–Fri). Each column scrolls independently. -->
            <div class="day-col" *ngFor="let day of daysVm">
                <div class="day-header">
                    <div class="day-name">{{ day.label }}</div>
                    <div class="day-date">{{ day.subLabel }}</div>
                </div>

                <div class="tiles">
                    <button kendoButton
                            *ngFor="let t of day.tiles"
                            class="slot-tile"
                            [ngClass]="{ 'selected': t.slotId === selectedSlotId }"
                            (click)="onTileClick(t)">
                        <div class="time">{{ t.timeLabel }}</div>
                        <div class="loc">{{ t.officeName }}</div>
                    </button>
                </div>
            </div>
        </div>

        <button kendoButton fillMode="flat" class="nav-btn next" (click)="goNextWeek()">Next week &rarr;</button>
    </div>

    <!-- Hold panel (unchanged behavior) -->
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

    private hhmmss(d: Date): string {
        const h = String(d.getHours()).padStart(2, '0');
        const m = String(d.getMinutes()).padStart(2, '0');
        return `${h}:${m}:00`;
    }

    ngOnDestroy(): void {
        this.destroy$.next();
        this.destroy$.complete();
    }
}
schedule-oath-confirm.component.html file:
<div class="container">
    <h2>Schedule Oath – Confirm Reservation</h2>

    <div class="hold-banner" [class.expired]="expired">
        Time remaining to hold appointment: <strong>{{ remaining }}</strong>
    </div>

    <kendo-card *ngIf="!expired">
        <kendo-card-body>
            <div class="grid-2">
                <div>
                    <label>Location</label>
                    <div>{{ slot?.officeName || '-' }}</div>
                </div>
                <div>
                    <label>Date</label>
                    <div>{{ startIso | date:'MM/dd/yyyy' }}</div>
                </div>
                <div>
                    <label>Start Time</label>
                    <div>{{ startIso | date:'shortTime' }}</div>
                </div>
                <div>
                    <label>End Time</label>
                    <div>{{ endIso | date:'shortTime' }}</div>
                </div>
            </div>

            <div class="disclaimer">
                <kendo-checkbox [(ngModel)]="disclaimerAccepted"></kendo-checkbox>
                <span>I acknowledge the disclaimer and agree to the terms.</span>
            </div>

            <button kendoButton themeColor="primary"
                    (click)="confirm()"
                    [disabled]="!disclaimerAccepted || expired">
                Confirm Reservation
            </button>
        </kendo-card-body>
    </kendo-card>

    <div *ngIf="expired" class="expired-note">
        Your hold has expired. Please go back and select another available slot.
        <div class="mt-12">
            <button kendoButton (click)="goBackToSlots()">Back to Slots</button>
        </div>
    </div>
</div>
schedule-oath-confirm.component.ts file: 
import { Component, NgZone, OnDestroy, OnInit } from '@angular/core';
import { Router } from '@angular/router';
import { interval, Subject, takeUntil } from 'rxjs';
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
app-routing.module.ts file: 
import { NgModule } from '@angular/core';
import { ExtraOptions, RouterModule, Routes } from '@angular/router';
import { LoginComponent } from './pages/login/login.component';
import { EnterpinComponent } from './pages/enterpin/enterpin.component';
import { AccountRegistrationComponent } from './pages/account-registration/account-registration.component';
import { EmailVerificationComponent } from './components/email-verification/email-verification.component';
import { PersonalInformationComponent } from './components/personal-information/personal-information.component';
import { AccountConfirmationComponent } from './components/account-confirmation/account-confirmation.component';
import { authGuard } from './guards/auth.guard';
import { SessionExpiredComponent } from './pages/session-expired/session-expired.component';
import { registrationAuthGuardGuard } from './guards/registration-guard/registration-auth-guard.guard';
import { ProfileHomepageComponent } from './pages/profile-homepage/profile-homepage.component';
import { NewAppPersonalInformationComponent } from './components/new-app-personal-information/new-app-personal-information.component';
import { NewAppAddressComponent } from './components/new-app-address/new-app-address.component';
import { NewAppApplicationInformationComponent } from './components/new-app-application-information/new-app-application-information.component';
import { NewAppReferencesComponent } from './components/new-app-references/new-app-references.component';
import { NewAppAdditionalInformationComponent } from './components/new-app-additional-information/new-app-additional-information.component';
import { NewAppApplicationReviewComponent } from './components/new-app-application-review/new-app-application-review.component';
import { NewAppAttestationComponent } from './components/new-app-attestation/new-app-attestation.component';
import { NewAppConfirmationComponent } from './components/new-app-confirmation/new-app-confirmation.component';
import { NotaryApplicationComponent } from './pages/notary-application/notary-application.component';
import { ScheduleOathComponent } from './pages/schedule-oath/schedule-oath.component';
import { ApplicationAddressGuard } from './guards/notary-application/application-address.guard';
import { UnauthorizedPageComponent } from './pages/unauthorized-page/unauthorized-page.component';
import { NotaryAcknowledgementComponent } from './components/remote-notary-acknowledgement/notary-acknowledgement.component';
import { ApplyNotaryApplicationComponent } from './pages/remote-apply-notary-application/apply-notary-application.component';
import { SelectPartnerComponent } from './components/remote-notary-partner/select-partner.component';
import { UpdateLoginInformationComponent } from './pages/update-login-information/update-login-information.component';
import { UpdateLoginConfirmComponent } from './components/update-login-confirm/update-login-confirm.component';
import { UpdateLoginNewEmailComponent } from './components/update-login-new-email/update-login-new-email.component';
import { UpdateLoginVerifyEmailComponent } from './components/update-login-verify-email/update-login-verify-email.component';
import { UpdateLoginInfoComponent } from './components/update-login-info/update-login-info.component';
import { RemoteConfirmationComponent } from './components/remote-notary-confirmation/remote-confirmation.component';
import { UpdateExternalProfileComponent } from './pages/update-external-profile/update-external-profile.component';
import { FindNotaryComponent } from './pages/find-notary/find-notary.component';
import { ScheduleOathPersonalInfoComponent } from './components/schedule-oath-personal-info/schedule-oath-personal-info.component';
import { ScheduleOathDateTimeComponent } from './components/schedule-oath-date-time/schedule-oath-date-time.component';
import { ScheduleOathGuard } from './guards/schedule-oath/schedule-oath.guard';
import { ScheduleOathConfirmComponent } from './components/schedule-oath-confirm/schedule-oath-confirm.component';
import { ScheduleOathConfirmationComponent } from './components/schedule-oath-confirmation/schedule-oath-confirmation.component';

const routerOptions: ExtraOptions = {
    scrollPositionRestoration: 'top', // <- this scrolls to top on navigation
    anchorScrolling: 'enabled',       // optional: for anchor (#id) scrolling
    scrollOffset: [0, 0],             // optional: x, y offset
};

const routes: Routes = [
    { path: '', redirectTo: '/login', pathMatch: 'full' },
    {
        path: 'login',
        component: LoginComponent,
        runGuardsAndResolvers: 'always',
        canActivate: [authGuard]  // Existing auth guard
    },
    {
        path: 'find-notary',
        component: FindNotaryComponent,
        runGuardsAndResolvers: 'always',
        canActivate: [authGuard]
    },
    {
        path: 'enterpin',
        component: EnterpinComponent,
        canActivate: [authGuard],
        runGuardsAndResolvers: 'always'
    },
    { path: 'session-expired', component: SessionExpiredComponent },
    { path: 'unauthorized', component: UnauthorizedPageComponent },
    {
        path: 'account-registration',
        component: AccountRegistrationComponent,
        children: [
            { path: '', redirectTo: 'email-verification', pathMatch: 'full' },
            {
                path: 'email-verification',
                component: EmailVerificationComponent,
                canActivate: [registrationAuthGuardGuard]
            },
            {
                path: 'personal-information',
                component: PersonalInformationComponent,
                canActivate: [registrationAuthGuardGuard]
            },
            {
                path: 'account-confirmation',
                component: AccountConfirmationComponent,
                canActivate: [registrationAuthGuardGuard]
            },
        ],
    },
    {
        path: 'profile-homepage',
        component: ProfileHomepageComponent,
        canActivate: [authGuard]
    },
    {
        path: 'update-profile',
        component: UpdateExternalProfileComponent,
        // canActivate: [authGuard]
    },

    {
        path: 'notary-application',
        component: NotaryApplicationComponent,
        canActivate: [authGuard],
        children: [
            { path: '', redirectTo: 'personal-information', pathMatch: 'full' },
            {
                path: 'personal-information',
                component: NewAppPersonalInformationComponent
            },
            {
                path: 'address',
                canActivate: [ApplicationAddressGuard],
                component: NewAppAddressComponent
            },
            {
                path: 'application-information',
                canActivate: [ApplicationAddressGuard],
                component: NewAppApplicationInformationComponent
            },
            {
                path: 'references',
                canActivate: [ApplicationAddressGuard],
                component: NewAppReferencesComponent
            },
            {
                path: 'additional-information',
                canActivate: [ApplicationAddressGuard],
                component: NewAppAdditionalInformationComponent
            },
            {
                path: 'application-review',
                canActivate: [ApplicationAddressGuard],
                component: NewAppApplicationReviewComponent
            },
            {
                path: 'attestation',
                canActivate: [ApplicationAddressGuard],
                component: NewAppAttestationComponent
            },
            {
                path: 'confirmation',
                canActivate: [ApplicationAddressGuard],
                component: NewAppConfirmationComponent
            },
        ],
    },
    { 
        path: 'remote-application',
        component: ApplyNotaryApplicationComponent,
        canActivate: [],
        children: [
            { path: '', redirectTo: 'notary-acknowledgement', pathMatch: 'full' },
            {
                path: 'notary-acknowledgement',
                component: NotaryAcknowledgementComponent
            },
            {
                path: 'select-partner',
                component: SelectPartnerComponent
            },
            {
                path: 'confirmation',
                component: RemoteConfirmationComponent
            }
        ]
    },
    {
        path: 'schedule-oath',
        component: ScheduleOathComponent,
        children: [
            { path: '', redirectTo: 'personal-information', pathMatch: 'full' },
            { path: 'personal-information', component: ScheduleOathPersonalInfoComponent },
            { path: 'date-time', component: ScheduleOathDateTimeComponent, canActivate: [ScheduleOathGuard] },
            { path: 'confirm', component: ScheduleOathConfirmComponent, canActivate: [ScheduleOathGuard] },
            { path: 'confirmation', component: ScheduleOathConfirmationComponent, canActivate: [ScheduleOathGuard] }
        ]
    },
    {
        path: 'hack-path',
        component: ScheduleOathConfirmationComponent
    },
    {
        path: 'update-login',
        component: UpdateLoginInformationComponent,
        canActivate: [authGuard],
        children: [
            { path: '', redirectTo: 'update-login-information', pathMatch: 'full' },
            {
                path: 'updateEmail',
                component: UpdateLoginInfoComponent,
          //      canActivate: [registrationAuthGuardGuard]
            },
            {
                path: 'enterEmail',
                component: UpdateLoginNewEmailComponent,
     //           canActivate: [registrationAuthGuardGuard]
            },
            {
                path: 'verifyEmail',
                component: UpdateLoginVerifyEmailComponent,
     //           canActivate: [registrationAuthGuardGuard]
            },
            {
                path: 'confirmEmail',
                component: UpdateLoginConfirmComponent
            }
        ],
    },
    { path: '**', redirectTo: '/session-expired', pathMatch: 'full' },
];

@NgModule({
    imports: [RouterModule.forRoot(routes)],
    exports: [RouterModule]
})
export class AppRoutingModule { }
service.ts file:
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
