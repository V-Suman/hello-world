Bug: There is a bug in the grid.. the bug is 2 fold.. firstly incorrect days are popping up on the column header. We should never see a sunday day on the 
calendar. We are seeing sunday. And the second aspect of the bug is.. when I click any reservation slot on the sunday and it console logs it.. 
the date in the console log is pointing to the Monday of that week. For example, in the screenshot I attached.. it has Sunday Sep 28 
and it has a slot at 5:00 AM. When I click that the console log is {
    "reservationSlotId": 812,
    "reservationId": 56,
    "office": {
        "officeId": 1,
        "value": "Boston"
    },
    "visitTypeId": 1,
    "scheduleDate": "2025-09-29",
    "startTime": "09:00:00",
    "endTime": "09:30:00"
}
If you notice the schedule date of that is 29th september. Which is incorrect. Fix this. 

Ask me any clarifying questions you have before you get started. 

html file: 
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
ts file: 
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

        const parseStart = (s: any): Date | null => {
            const utc = s.startTimeUtc || s.startUtc;
            if (utc) {
                const d = new Date(utc);
                return Number.isNaN(d.getTime()) ? null : d;
            }
            const sd = s.scheduleDate, st = s.startTime;
            if (sd && st) {
                const guess = new Date(`${sd}T${st}Z`);
                return Number.isNaN(guess.getTime()) ? null : guess;
            }
            return null;
        };

        return rawSlots
            .map(s => {
                const start = parseStart(s);
                if (!start) return null;

                const slotId = Number(s.reservationSlotId ?? s.slotId ?? s.id ?? 0);
                const officeId = s.office?.officeId ?? s.officeId ?? null;
                const officeName = s.office?.value ?? officeNameById(officeId);

                return {
                    slotId,
                    start,
                    timeLabel: this.formatTime(start),
                    officeId,
                    officeName,
                    raw: s
                } as TileVM;
            })
            .filter((x): x is TileVM => !!x)
            .sort((a, b) => {
                const t = a.start.getTime() - b.start.getTime();
                if (t !== 0) return t;
                return a.officeName.localeCompare(b.officeName);
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
        // Build 5 days starting at server week start
        const start = new Date(weekStartIso);
        const days: DayVM[] = Array.from({ length: 5 }).map((_, i) => {
            const d = new Date(start);
            d.setUTCDate(start.getUTCDate() + i);
            const key = d.toISOString().slice(0, 10); // YYYY-MM-DD
            return {
                key,
                label: d.toLocaleDateString(undefined, { weekday: 'long' }),
                subLabel: d.toLocaleDateString(undefined, { month: 'short', day: 'numeric' }),
                tiles: []
            };
        });

        // Group tiles by date key
        const byKey = new Map<string, TileVM[]>();
        for (const t of tiles) {
            const k = t.start.toISOString().slice(0, 10);
            if (!byKey.has(k)) byKey.set(k, []);
            byKey.get(k)!.push(t);
        }

        for (const d of days) {
            d.tiles = (byKey.get(d.key) || []).slice(); // already sorted
        }

        return days;
    }

    // ---------------- interactions ----------------
    onTileClick(t: TileVM): void {
        //  Console-log it in backend response shape
        const slotPayload = {
            reservationSlotId: t.raw?.reservationSlotId ?? t.slotId,
            reservationId: t.raw?.reservationId ?? 0,
            office: {
                officeId: t.officeId ?? 0,
                value: t.officeName
            },
            visitTypeId: t.raw?.visitTypeId ?? 0,
            scheduleDate: t.raw?.scheduleDate ?? t.start.toISOString().slice(0, 10),
            startTime: t.raw?.startTime ?? this.formatTime(t.start),
            endTime: t.raw?.endTime ?? ''
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
            const s = new Date(startIso);
            const e = new Date(endIso);

            const fmt = (d: Date) =>
                d.toLocaleDateString(undefined, {
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

    ngOnDestroy(): void {
        this.destroy$.next();
        this.destroy$.complete();
    }
}
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
