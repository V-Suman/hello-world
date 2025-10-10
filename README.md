Bug: When the user selects a radio button and clicks next.. the criteria for the user to cross the page is done.. so the nav-bar section 
on the parent component schedule-oath.component should show the icon done.svg instead of v1.svg and the next step's icon should change 
from uv2.svg to v2.svg and so should its text to bold styling. That is not happening. 

Details: When I land on the parent page (with the first child page already rendered in the details section) this is how the URL looks like 
http://localhost:4200/schedule-oath/personal-information?applicantId=7&email=test01@oathdue.com now.. when I click the next button by selecting 
any radio button.. it should change the nav-bar section of the parent to the relevant stuff. It is not happening. 

Ask me any clarifying questions you have before you proceed. 
parent schedule-oath.component.html: 
<div class="oath-scheduler-page">
    <div class="nav-pane">
        <div class="side-bar-heading">Oath Scheduler</div>

        <div class="pages-list" *ngIf="stepState as s">
            <div class="step personal-information">
                <img [src]="s.personalInformation.icon" alt="Personal Information" class="icon" />
                <span class="step-link" [ngClass]="s.personalInformation.textClass">Mode of Oath</span>
            </div>

            <div class="step date-time">
                <img [src]="s.dateTime.icon" alt="Date & Time" class="icon" />
                <span class="step-link" [ngClass]="s.dateTime.textClass">Date &amp; Time</span>
            </div>

            <div class="step confirm-reservation">
                <img [src]="s.confirmReservation.icon" alt="Confirm Reservation" class="icon" />
                <span class="step-link" [ngClass]="s.confirmReservation.textClass">Confirm Reservation</span>
            </div>

            <div class="step reservation-confirmation">
                <img [src]="s.reservationConfirmation.icon" alt="Reservation Confirmation" class="icon" />
                <span class="step-link" [ngClass]="s.reservationConfirmation.textClass">Reservation Confirmation</span>
            </div>
        </div>
    </div>

    <div class="details-area">
        <div #notificationContainer [ngClass]="notificationContainerClass"></div>
        <router-outlet></router-outlet>
    </div>
</div>
parent schedule-oath.component.ts file:
import { Component, OnDestroy, OnInit, ViewChild, ViewContainerRef } from '@angular/core';
import { Router, NavigationEnd, NavigationStart } from '@angular/router';
import { filter, Subscription } from 'rxjs';
import { NotificationRef, NotificationService } from '@progress/kendo-angular-notification';
import { OathStepState, OathStepTrackerService } from '../../services/helper-services/oath-schedule-step-tracker/oath-step-tracker.service';


@Component({
    selector: 'app-schedule-oath',
    templateUrl: './schedule-oath.component.html',
    styleUrl: './schedule-oath.component.css'
})
export class ScheduleOathComponent implements OnInit, OnDestroy {
    @ViewChild('notificationContainer', { read: ViewContainerRef, static: true }) notificationContainer!: ViewContainerRef;

    stepState!: OathStepState;
    notificationContainerClass = 'notification-component';
    private routeSub!: Subscription;
    private stepSub!: Subscription;
    private childNotificationSub!: Subscription;
    private lastNotificationRef: NotificationRef | null = null;

    constructor(
        private router: Router,
        private notificationService: NotificationService,
        private stepTracker: OathStepTrackerService
    ) { }

    ngOnInit(): void {
        // Bridge child -> parent notifications
        this.childNotificationSub = this.stepTracker.childNotification$.subscribe(n => {
            if (!n) return;
            this.triggerNotification(n.message, n.className);
        });

        // Keep a local copy of step state for template
        this.stepSub = this.stepTracker.stepState$.subscribe(s => this.stepState = s);

        // Clear notifications on nav start
        this.router.events.pipe(filter(e => e instanceof NavigationStart))
            .subscribe(() => {
                if (this.notificationContainer) this.notificationContainer.clear();
                if (this.lastNotificationRef) { this.lastNotificationRef.hide(); this.lastNotificationRef = null; }
            });

        // Update step visuals per current child route
        this.routeSub = this.router.events.pipe(
            filter((e): e is NavigationEnd => e instanceof NavigationEnd)
        ).subscribe(e => {
            const seg = e.url.split('/').pop() || '';
            // style hook; optional per route
            if (seg.includes('personal-information')) this.notificationContainerClass = 'notification-component personal-information';
            else if (seg.includes('date-time')) this.notificationContainerClass = 'notification-component date-time';
            else if (seg.includes('confirm')) this.notificationContainerClass = 'notification-component confirm';
            else if (seg.includes('confirmation')) this.notificationContainerClass = 'notification-component confirmation';
            else this.notificationContainerClass = 'notification-component';

            this.stepTracker.updateStatesForRoute(seg);
        });
    }

    private triggerNotification(message: string, className: string): void {
        if (this.lastNotificationRef) this.lastNotificationRef.hide();
        this.lastNotificationRef = this.notificationService.show({
            content: message,
            cssClass: className,
            animation: { type: 'fade', duration: 200 },
            hideAfter: 200,
            position: { horizontal: 'center', vertical: 'top' },
            closable: true,
            width: 500,
            appendTo: this.notificationContainer,
        });
    }

    ngOnDestroy(): void {
        if (this.routeSub) this.routeSub.unsubscribe();
        if (this.stepSub) this.stepSub.unsubscribe();
        if (this.childNotificationSub) this.childNotificationSub.unsubscribe();
    }
}
oath-step-tracker.service.ts file:
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';

export interface StepStyle {
    icon: string;
    textClass: string;
}

export interface OathStepState {
    personalInformation: StepStyle;
    dateTime: StepStyle;
    confirmReservation: StepStyle;
    reservationConfirmation: StepStyle;
}

export interface ChildNotification {
    message: string;
    className: string;
}

@Injectable({ providedIn: 'root' })
export class OathStepTrackerService {
    // Icons and classes (reuse names from registration)
    private readonly ICONS = {
        v1: `assets/v1.svg`,
        v2: `assets/v2.svg`,
        v3: `assets/v3.svg`,
        v4: `assets/v4.svg`,
        uv2: `assets/uv2.svg`,
        uv3: `assets/uv3.svg`,
        uv4: `assets/uv4.svg`,
        done: `assets/done.svg`,
    };

    private readonly STYLES = {
        activeBold: 'step-text-active',
        inactive: 'step-text-inactive',
        completed: 'step-text-completed',
    };

    private stepStateSource = new BehaviorSubject<OathStepState>(this.initial());
    stepState$ = this.stepStateSource.asObservable();

    // Child -> parent notifications (optional, like registration)
    private childNotificationSubject = new BehaviorSubject<ChildNotification | null>(null);
    childNotification$ = this.childNotificationSubject.asObservable();

    triggerChildNotification(message: string, className: string): void {
        this.childNotificationSubject.next({ message, className });
    }

    /** Deterministic route-based mapping (implements your rule #6). */
    updateStatesForRoute(routeSegment: string): void {
        switch (routeSegment) {
            case 'personal-information':
                this.stepStateSource.next({
                    personalInformation: { icon: this.ICONS.v1, textClass: this.STYLES.activeBold },
                    dateTime: { icon: this.ICONS.uv2, textClass: this.STYLES.inactive },
                    confirmReservation: { icon: this.ICONS.uv3, textClass: this.STYLES.inactive },
                    reservationConfirmation: { icon: this.ICONS.uv4, textClass: this.STYLES.inactive },
                });
                break;

            case 'date-time':
                this.stepStateSource.next({
                    personalInformation: { icon: this.ICONS.done, textClass: this.STYLES.completed },
                    dateTime: { icon: this.ICONS.v2, textClass: this.STYLES.activeBold },
                    confirmReservation: { icon: this.ICONS.uv3, textClass: this.STYLES.inactive },
                    reservationConfirmation: { icon: this.ICONS.uv4, textClass: this.STYLES.inactive },
                });
                break;

            case 'confirm':
            case 'confirm-reservation':
                this.stepStateSource.next({
                    personalInformation: { icon: this.ICONS.done, textClass: this.STYLES.completed },
                    dateTime: { icon: this.ICONS.done, textClass: this.STYLES.completed },
                    confirmReservation: { icon: this.ICONS.v3, textClass: this.STYLES.activeBold },
                    reservationConfirmation: { icon: this.ICONS.uv4, textClass: this.STYLES.inactive },
                });
                break;

            case 'confirmation':
                this.stepStateSource.next({
                    personalInformation: { icon: this.ICONS.done, textClass: this.STYLES.completed },
                    dateTime: { icon: this.ICONS.done, textClass: this.STYLES.completed },
                    confirmReservation: { icon: this.ICONS.done, textClass: this.STYLES.completed },
                    reservationConfirmation: { icon: this.ICONS.v4, textClass: this.STYLES.activeBold },
                });
                break;

            default:
                // keep initial
                this.stepStateSource.next(this.initial());
                break;
        }
    }

    /** Called by children on the action that truly marks completion. */
    completePersonalAndActivateDateTime(): void {
        this.stepStateSource.next({
            personalInformation: { icon: this.ICONS.done, textClass: this.STYLES.completed },
            dateTime: { icon: this.ICONS.v2, textClass: this.STYLES.activeBold },
            confirmReservation: { icon: this.ICONS.uv3, textClass: this.STYLES.inactive },
            reservationConfirmation: { icon: this.ICONS.uv4, textClass: this.STYLES.inactive },
        });
    }

    completeDateTimeAndActivateConfirm(): void {
        this.stepStateSource.next({
            personalInformation: { icon: this.ICONS.done, textClass: this.STYLES.completed },
            dateTime: { icon: this.ICONS.done, textClass: this.STYLES.completed },
            confirmReservation: { icon: this.ICONS.v3, textClass: this.STYLES.activeBold },
            reservationConfirmation: { icon: this.ICONS.uv4, textClass: this.STYLES.inactive },
        });
    }

    completeConfirmAndActivateFinal(): void {
        this.stepStateSource.next({
            personalInformation: { icon: this.ICONS.done, textClass: this.STYLES.completed },
            dateTime: { icon: this.ICONS.done, textClass: this.STYLES.completed },
            confirmReservation: { icon: this.ICONS.done, textClass: this.STYLES.completed },
            reservationConfirmation: { icon: this.ICONS.v4, textClass: this.STYLES.activeBold },
        });
    }

    reset(): void {
        this.stepStateSource.next(this.initial());
        this.childNotificationSubject.next(null);
    }

    private initial(): OathStepState {
        return {
            personalInformation: { icon: this.ICONS.v1, textClass: this.STYLES.activeBold },
            dateTime: { icon: this.ICONS.uv2, textClass: this.STYLES.inactive },
            confirmReservation: { icon: this.ICONS.uv3, textClass: this.STYLES.inactive },
            reservationConfirmation: { icon: this.ICONS.uv4, textClass: this.STYLES.inactive },
        };
    }
}
The first child component schedule-oath-personal-information.component.ts file: 
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
        const order: VisitCode[] = ['InPerson', 'External', 'Virtual'];
        return opts.sort((a, b) => order.indexOf(a.code) - order.indexOf(b.code));
    }

    private defaultVisitOptions(): VisitVM[] {
        return [
            { id: 1, code: 'InPerson', label: 'In Person', help: 'at a location closer to you' },
            { id: 2, code: 'External', label: 'External', help: 'I will find my own commissioner to qualify' },
            { id: 3, code: 'Virtual', label: 'Virtual', help: 'with a virtual oath you maybe able to schedule an oath at any location' }
        ];
    }

    ngOnDestroy(): void {
        this.destroy$.next();
        this.destroy$.complete();
    }
}
