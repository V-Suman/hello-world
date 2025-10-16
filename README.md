Issue: When the schedule-oath-confirm component loads up.. the slot hold timer is already running.. depending on how 
long the user took to interact with the kendo-dialog in the schedule-oath-date-time component. We want to display the 
time remaining on the kendo-dialog box as well.. so that the user knows that their timer has started. 

Details: When the user clicks on a tile in schedule-oath-date-time component we throw up a kendo-dialogbox and start 
the countdown timer in the background but not display it to the user until the user lands on the next page.. by the 
time that happens some time has already elapsed. Instead why don't we just show the timer on the kendo-dialog also
that way there is uniformity. 

Remember.. the logic in the schedule-oath-confirm component stays as is.. we just want to add the countdown timer 
to the kendo-dialogbox as well. Just a small line which shows countdown that winds down. 

Ask me clarifying questions before you get started 

step 2 date and time html file: 
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
                                (valueChange)="onOfficeFilterChange($event)"
                                [disabled]="isVirtualMode">
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
    <!--<div class="hold" *ngIf="hold as h">
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
                    (click)="goToConfirm()">
                Confirm Reservation
            </button>
        </div>
    </div>-->
    <kendo-dialog *ngIf="dialogVisible" (close)="closeDialog()" title=" " [width]="480">
        <ng-container>
            <div class="dialog-header">
                <h3>Slot Selection Confirmation</h3>
                <button class="dialog-close" (click)="closeDialog()">X</button>
            </div>
            <div class="dialog-body">
                <div class="disclaimer">
                    <p>Appointment ID: <strong>{{ hold?.appointmentId }}</strong></p>
                    <div class="input-and-label">
                        <input type="checkbox" [(ngModel)]="acceptedDisclaimer" />
                        I understand this hold will be released if I don't confirm promptly.
                    </div>
                </div>
            </div>
            <div class="dialog-actions">
                <button kendoButton class="release-hold" (click)="closeDialog()">Release Hold</button>
                <button kendoButton class="confirm-reservation" [disabled]="!acceptedDisclaimer || confirming" (click)="goToConfirm()">Confirm Reservation</button>
            </div>
        </ng-container>
    </kendo-dialog>

</div>

<ng-template #busy>
    <div class="busy">
        <kendo-loader type="infinite-spinner"></kendo-loader>
        <div>Loading available offices and slots...</div>
    </div>
</ng-template>
date and time ts file: 
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
        const visitMode = (qp.get('visitMode') as any) || this.store.snapshot.visitMode || 'inPerson';
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
            visitTypeId: this.visitTypeId, visitMode 
        });

        this.loadOffices(() => this.loadSlotsWithFallback());
    }

    // ---------------- data loads ----------------
    private loadOffices(onDone?: () => void) {
        this.svc.getOffices().pipe(takeUntil(this.destroy$)).subscribe({
            next: (offices) => {
                const all = (offices || []).filter(o => (o as any).isActive ?? true);

                if (this.isVirtualMode) {
                    // find the virtual office by value and/or id
                    const virtual = all.find(o => (o as any).value === 'Virtual' || o.officeId === 4);
                    if (!virtual) {
                        alert('Virtual scheduling is not available right now. Please choose another option.');
                        this.router.navigate(['/schedule-oath/personal-information'], { queryParamsHandling: 'preserve' });
                        return;
                    }
                    this.offices = [virtual];
                    this.selectedOfficeId = virtual.officeId;
                    this.selectedOfficeIdFilter = virtual.officeId;

                    // single, disabled dropdown
                    this.officeFilterOptions = [{ text: (virtual as any).value ?? 'Virtual', value: virtual.officeId }];
                } else {
                    // In-person or external → hide Virtual
                    this.offices = all.filter(o => ((o as any).value ?? '').toLowerCase() !== 'virtual');

                    this.officeFilterOptions = [
                        { text: 'All locations', value: null },
                        ...this.offices.map(o => ({ text: (o as any).value ?? o.name ?? `Office ${o.officeId}`, value: o.officeId }))
                    ];

                    if (!this.selectedOfficeId && this.offices.length) {
                        this.selectedOfficeId = this.offices[0].officeId;
                    }
                }
            },
            error: () => { },
            complete: () => { if (onDone) onDone(); this.cdr.markForCheck(); }
        });
    }

    public get isVirtualMode(): boolean {
        // prefer query param then store (supports deep link/back)
        const qpMode = this.route.snapshot.queryParamMap.get('visitMode') as any;
        const mode = (qpMode || this.store.snapshot.visitMode) as ('inPerson' | 'virtual' | 'external' | undefined);
        return mode === 'virtual';
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
step 3 confirm component html file: 
<div class="container">
    <div class="hold-banner" [class.expired]="expired">
        Time remaining to hold appointment: <strong>{{ remaining }}</strong>
    </div>
    <div class="confirm-reservation-heading">Confirm your reservation</div>
    <div class="not-quite">Your appointment is not quite done. Please read and confirm the details below.</div>
    <!--<kendo-card *ngIf="!expired">
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
    </kendo-card>-->
    <div class="black-card" *ngIf="!expired">
        <div class="location">
            <div class="location-label"><strong>Location</strong></div>
            <div class="location-data">{{ modeOptionTrailingText }}</div>
        </div>
        <div class="date">
            <div class="date-label"><strong>Date and time</strong></div>
            <div class="date-data">
                <div>{{ startIso | date:'EEEE, MMMM d, y' }}</div>
                <div *ngIf="endIso; else startOnly">
                    {{ startIso | date:'h:mm a' }} – {{ endIso | date:'h:mm a' }}
                </div>
                <ng-template #startOnly>
                    <div>{{ startIso | date:'h:mm a' }}</div>
                </ng-template>
            </div>
        </div>
    </div>

    <div class="blue-card" *ngIf="!expired">
        <div class="disclaimer-paragraph">
            <p><strong>Please review and agree to the following terms before submitting your reservation request</strong></p>
            <ol *ngIf="mode === 'inPerson'">
                <li> Please have a copy of your reservation confirmation, proof of identification, and official notary application approval letter</li>
                <li>
                    For In-person reservations arrive 5 minutes prior to your reservation time. If you arrive more than 5 minutes AFTER your reservation time,
                    you may be rescheduled or experience delays in getting your oath administered.
                </li>
            </ol>
            <ol *ngIf="mode === 'virtual'">
                <li> Please have a copy of your reservation confirmation and proof of identification for verification purposes</li>
                <li>
                    For Virtual reservations you may benefit with launching the application a few minutes prior to the scheduled appointment time to ensure 
                    your camera and microphone are working appropriately.
                </li>
            </ol>
        </div>
        <div class="disclaimer">
            <kendo-checkbox [(ngModel)]="disclaimerAccepted"></kendo-checkbox>
            <span>I have read, understood and agree to the above terms</span>
        </div>
    </div>

    <div class="buttons-row">
        <button kendoButton
                (click)="goBackToSlots()"
                class="previous-button">
            Previous
        </button>
        <button kendoButton
                (click)="confirm()"
                class="confirm-reservation"
                [disabled]="!disclaimerAccepted || expired">
            Confirm Reservation
        </button>
    </div>

    <div *ngIf="expired" class="expired-note">
        Your hold has expired. Please go back and select another available slot.
        <div class="mt-12">
            <button kendoButton (click)="goBackToSlots()">Back to Slots</button>
        </div>
    </div>
</div>
step3 confirm component ts file: 
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
    modeOptionTrailingText = '';
    mode = 'inPerson';

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
        this.mode = (this.store.snapshot.visitMode) ? (this.store.snapshot.visitMode) : ''; // 'inPerson' | 'virtual' | 'external'
        this.modeOptionTrailingText =
            (this.mode === 'inPerson') ?
            (s.selectedSlot.officeName +' '+'(In-Person reservation)') : 
            ('Virtual Appointment');

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

        if (!s.visitTypeId) { this.goBackToSlots(); return; }

        this.service.confirmReservation({
            visitTypeId: s.visitTypeId,
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
