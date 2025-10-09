1.1 I will post the app-routing.module.ts file 
1.2 I will post the app-routing.moduel.ts file
2.1 I will post the existing code I have for the oath-scheduler 
parent page 
2.2 No I don't 
3.1 I will post the code for those as well 
3.2 No. They should not be clickable. They are read-only indicators
4.1 Yes same 3 states. Active, Visited and Completed. YOu can use the same 
asset paths. For 4th Reservation Confirmation state you can 
just do assets/v4.svg, assets/uv4.svg 
4.2 Re-use the same ones. 
5 Actually for now.. we will make the next button click of each 
step as completion. I will worry about the completion rules later. 
6. If the user is on step 2 that means step 1 is completed and when 
the user goes back to step 1 we make the step 2 as unvisited.. 
then make the step 1 as in progress. Makes sense? 
7. Use a dedicated service for that. 
8. Don't worry about that for now. 
9. Yes. I want a notification container. 
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
schedule-oath.component.html file: 
<p>Oath Scheduler will use this page</p>
schedule-oath.component.ts file: 
import { Component } from '@angular/core';

@Component({
  selector: 'app-schedule-oath',
  templateUrl: './schedule-oath.component.html',
  styleUrl: './schedule-oath.component.css'
})
export class ScheduleOathComponent {

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
schedule-oath-personal-info.component.html file:
<section class="so-personal-info">
    <!-- Non-blocking skeleton -->
    <div class="skeleton" *ngIf="loading" aria-hidden="true">Loading…</div>

    <form *ngIf="!loading" [formGroup]="form" (ngSubmit)="next()" novalidate>
        <h1 class="page-title">Personal Information</h1>

        <div class="profile-card" *ngIf="profile as p">
            <div class="profile-line">
                <strong>Name:</strong> {{ p.fullName }}
            </div>

            <div class="profile-line" *ngIf="store.snapshot.email">
                <strong>Email:</strong> {{ store.snapshot.email }}
            </div>
        </div>

        <fieldset class="visit-types">
            <legend>Choose Visit Type</legend>

            <div class="options">
                <div class="option" *ngFor="let opt of visitOptions">
                    <!-- Unique id; keep the same 'name' for grouping -->
                    <input type="radio"
                           [id]="'visitTypeId-' + opt.id"
                           name="visitTypeId"
                           formControlName="visitTypeId"
                           [value]="'' + opt.id"             
                    (change)="onSelectVisitType($event)"
                    />
                    <label class="text"
                           [for]="'visitTypeId-' + opt.id"
                           (click)="form.get('visitTypeId')?.setValue('' + opt.id)"  
          >
            <div class="title">{{ opt.label }}</div>
            <div class="help" *ngIf="opt.help">{{ opt.help }}</div>
          </label>
                </div>
            </div>

            <div class="error" *ngIf="submitted && form.get('visitTypeId')?.invalid">
                Please select a visit type to continue.
            </div>
        </fieldset>

        <div class="actions">
            <button type="button" class="btn secondary" (click)="cancel()">Cancel</button>
            <button type="submit" class="btn primary">Next</button>
        </div>
    </form>
</section>
schedule-oath-date-time.component.html file:
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
schedule-oath-confirm-reservation.component.html file: 
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
schedule-oath-confirm-reservation.component.ts file:
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

        // Must have an appointmentId to confirm
        if (!s.appointmentId) { this.goBackToSlots(); return; }

        this.service.confirmReservation({
            appointmentId: s.appointmentId,
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
schedule-oath-confirmation.component.html file: 
<div class="container" *ngIf="store.snapshot.confirmation as conf; else missing">
    <h2>Reservation Confirmation</h2>
    <kendo-card>
        <kendo-card-body>
            <div class="grid-2">
                <div>
                    <label>Confirmation #</label>
                    <div class="conf">{{ conf.confirmationNumber }}</div>
                </div>
                <div>
                    <label>Location</label>
                    <div>{{ conf.officeName }}</div>
                </div>
                <div>
                    <label>Date</label>
                    <div>{{ conf.startTimeUtc | date:'MM/dd/yyyy' }}</div>
                </div>
                <div>
                    <label>Time</label>
                    <div>{{ conf.startTimeUtc | date:'shortTime' }} – {{ conf.endTimeUtc | date:'shortTime' }}</div>
                </div>
            </div>
            <div class="actions">
                <button kendoButton (click)="print()">Print</button>
                <button kendoButton themeColor="primary" (click)="done()">Done</button>
            </div>
        </kendo-card-body>
    </kendo-card>
</div>

<ng-template #missing>
    <div class="container">
        <p>Missing confirmation details. Please start again.</p>
        <button kendoButton (click)="startOver()">Start Over</button>
    </div>
</ng-template>
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
