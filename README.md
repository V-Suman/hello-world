Goal: To add code to the existing oath-scheduler parent page that should then get 
the ability to track the states of the child pages and change the logos of the steps in the parent page. 

More Details: The idea is that.. the oath-scheduler page will have 2 sections the left side nav-bar 
section which has the child pages names, icons for said names and the right side section which is 
essentially just a container where all the child pages are rendered. 

The oath-scheduler page should behave like a dumb container which can only keep a track of which step 
the user is currently in. 

Let’s say if the user is on step - 1 which is the personal information section.. the personal 
information heading in the left side nav-bar section should follow a certain styling (bold letters) 
and the icon being displayed should be different. If the user has finished the work on that page and 
clicked next.. the styling on the page header in the nav-bar should change and so should the icon. 
This is the general process. When the user hits the “previous” button on the step - 2 page. 
The step - 1 page should load and have the relevant styling and icons. 

The holistic process is as follows: 
The user lands on the localhost:4200/oath-scheduler and the OathSchedulerPersonalInformation component
should be the initial one being rendered in the content section. 

Initial State and Followup logic: 
Left Section- Should have 4 step texts for 4 pages Personal Information, Date and Time, Confirm Reservation, 
Reservation Confirmation with the Personal Information section highlighted. and relevant Icon displayed. 
When the user does some job on the page and they satisfy the conditon to move forward and the user clicks 
next.. we then change the state of the side nav-bar items such that the step - 1 personal information's 
icon is now a tick icon (available in my assests folder) and step - 2 now has active step related styling
meaning step text is bold and the icon is different. So on an so forth. If for some reason the user 
has decided to go back to step 1 while theuy are in step 2 we allow them.. but the look and feel of it is 
different. 

Task at hand: 
I already implemented this in some other page.. what I want you to do is to implement similar logic here. 
I will paste the component in which I implemented this to give you a better understanding of how I did it. 
I want you to figure out how I did it.. then use that knowledge to draft me a plan to help me implement it 
in my oath-scheduler mini app. Cool? 

Ask me any clarifying questions you might have before you give me a plan. Then I will give you the oath-scheduler 
individual steps components code.. then you give me the code to implement it. 

The pictures I attached shows how it should look like and how It should behave. 

Here is another mini app inside my website that I implemented what I'm talking about: 
account-registration.component.ts file: 
import { Component, OnInit, OnDestroy, ViewChild, ViewContainerRef } from '@angular/core';
import { Router, NavigationEnd } from '@angular/router';
import { Subscription } from 'rxjs';
import { filter } from 'rxjs/operators';
import { RegistrationStepTrackerService, StepState } from '../../services/helper-services/registration-step-tracker/registration-step-tracker.service';
import { RegistrationFlowTokenService } from '../../services/registration-services/registration-token-service/registration-flow-token.service';
import { NotificationRef, NotificationService } from '@progress/kendo-angular-notification';
import { NavigationStart } from '@angular/router';

@Component({
    selector: 'app-account-registration',
    templateUrl: './account-registration.component.html',
    styleUrls: ['./account-registration.component.css']
})
export class AccountRegistrationComponent implements OnInit, OnDestroy {
    @ViewChild('notificationContainer', { read: ViewContainerRef, static: true }) notificationContainer!: ViewContainerRef;
    private lastNotificationRef: NotificationRef | null = null;
    notificationContainerClass: string = 'notification-component';
    private childNotificationSub!: Subscription;
    stepState!: StepState;
    private routeSub!: Subscription;
    private otpWarningSub!: Subscription;
    private otpExpiredSub!: Subscription;
    private sessionWarningSub!: Subscription;
    private sessionExpiredSub!: Subscription;
    private expirationCheckInterval!: any;

    constructor(
        private router: Router,
        private stepTrackerService: RegistrationStepTrackerService,
        private registrationFlowTokenService: RegistrationFlowTokenService,
        private notificationService: NotificationService
    ) { }

    ngOnInit(): void {
        this.childNotificationSub = this.stepTrackerService.childNotification$.subscribe(notification => {
            if (notification) {
                this.triggerNotification(notification.message, notification.className);
            }
        });

        this.stepTrackerService.stepState$.subscribe(state => {
            this.stepState = state;
        });

        this.router.events.pipe(
            filter(event => event instanceof NavigationStart)
        ).subscribe((event) => {
            // Clear any active notifications before navigation begins.
            if (this.notificationContainer) {
                this.notificationContainer.clear();
            }
            if (this.lastNotificationRef) {
                this.lastNotificationRef.hide();
                this.lastNotificationRef = null;
            }
        });

        this.routeSub = this.router.events.pipe(
            filter((event): event is NavigationEnd => event instanceof NavigationEnd)
        ).subscribe((event) => {
            const currentRoute = event.url;

            if (currentRoute.includes('email-verification')) {
                this.notificationContainerClass = 'notification-component email-verification';
            } else if (currentRoute.includes('personal-information')) {
                this.notificationContainerClass = 'notification-component personal-information';
            } else {
                this.notificationContainerClass = 'notification-component';
            }

            // Clear sessionStorage and reset flags for specific routes
            if (currentRoute.includes('session-expired') || currentRoute.includes('login')) {
                sessionStorage.removeItem('registrationOtpExpiryTime');
                sessionStorage.removeItem('registrationFlowExpiresAt');
                this.stepTrackerService.resetFlags();
            }

            // Reset service flags when navigating to email-verification
            if (currentRoute.includes('email-verification')) {
                this.stepTrackerService.resetFlags();
            } else {
                // When not on email-verification, reset the OTP state
                this.stepTrackerService.resetOtpState();
            }

            // Update step navigation (extract the last route segment)
            const routeSegment = currentRoute.split('/').pop();
            if (routeSegment) {
                this.stepTrackerService.updateStatesForRoute(routeSegment);
            }

            // Track expirations (now, without an OTP expiry value, OTP notifications won’t fire)
            this.stepTrackerService.trackExpirations();
        });

        this.otpWarningSub = this.stepTrackerService.otpWarning$.subscribe(message => {
            if (message) {
                this.triggerNotification(message, 'custom-notification info');
            }
        });

        this.otpExpiredSub = this.stepTrackerService.otpExpired$.subscribe(message => {
            if (message) {
                this.triggerNotification(message, 'custom-notification incorrect');
            }
        });

        this.sessionWarningSub = this.stepTrackerService.sessionWarning$.subscribe(message => {
            if (message) {
                this.triggerNotification(message, 'custom-notification info');
            }
        });

        this.sessionExpiredSub = this.stepTrackerService.sessionExpired$.subscribe(message => {
            if (message) {
                this.triggerNotification(message, 'custom-notification incorrect');
                this.stepTrackerService.resetFlags();
                this.registrationFlowTokenService.clearToken();
                this.router.navigate(['/session-expired']);
            }
        });

        this.expirationCheckInterval = setInterval(() => {
            this.stepTrackerService.trackExpirations();
        }, 5000);
    }

    triggerNotification(notificationMessage: string, className: string): void {

        if (this.lastNotificationRef) { this.lastNotificationRef.hide(); }

        this.lastNotificationRef = this.notificationService.show({
            content: notificationMessage,
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
        if (this.otpWarningSub) this.otpWarningSub.unsubscribe();
        if (this.otpExpiredSub) this.otpExpiredSub.unsubscribe();
        if (this.sessionWarningSub) this.sessionWarningSub.unsubscribe();
        if (this.sessionExpiredSub) this.sessionExpiredSub.unsubscribe();
        if (this.expirationCheckInterval) {
            clearInterval(this.expirationCheckInterval);
        }
        if (this.childNotificationSub) { this.childNotificationSub.unsubscribe(); }
    }
}
account-registration.component.html file: 
<div class="account-registration-page">
    <div class="nav-pane">
        <div class="side-bar-heading">Account Registration</div>
        <div class="pages-list">
            <div class="email-verification">
                <img [src]="stepState.emailVerification.icon" alt="Email verification" class="icon" />
                <a class="step-link" [ngClass]="stepState.emailVerification.textClass">
                    Email Verification
                </a>
            </div>
            <div class="personal-information">
                <img [src]="stepState.personalInformation.icon" alt="Personal Information verification" class="icon" />
                <a class="step-link" [ngClass]="stepState.personalInformation.textClass">
                    Personal Information
                </a>
            </div>
            <div class="account-confirmation">
                <img [src]="stepState.accountConfirmation.icon" alt="Account Information verification and confirmation" class="icon" />
                <a class="step-link" [ngClass]="stepState.accountConfirmation.textClass">
                    Account Confirmation
                </a>
            </div>
        </div>
    </div>
    <div class="details-area">
        <div #notificationContainer [ngClass]="notificationContainerClass"></div><router-outlet></router-outlet>
    </div>
</div>
The service I used to track steps registration-step-tracker.service.ts file: 
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';

export interface StepStyle {
    icon: string;
    textClass: string;
}

export interface StepState {
    emailVerification: StepStyle;
    personalInformation: StepStyle;
    accountConfirmation: StepStyle;
}

export interface ChildNotification {
    message: string;
    className: string;
}

@Injectable({
    providedIn: 'root'
})
export class RegistrationStepTrackerService {
    private emailVerified = false;
    private personalInfoCompletion = false;

    private readonly ICONS = {
        visited1: `assets/v1.svg`,
        visited2: `assets/v2.svg`,
        unvisited2: `assets/uv2.svg`,
        unvisited3: `assets/uv3.svg`,
        done: `assets/done.svg`
    };

    private readonly STYLES = {
        activeBold: 'step-text-active',
        inactive: 'step-text-inactive',
        completed: 'step-text-completed'
    };

    private stepStateSource = new BehaviorSubject<StepState>({
        emailVerification: {
            icon: this.ICONS.visited1,
            textClass: this.STYLES.activeBold
        },
        personalInformation: {
            icon: this.ICONS.unvisited2,
            textClass: this.STYLES.inactive
        },
        accountConfirmation: {
            icon: this.ICONS.unvisited3,
            textClass: this.STYLES.inactive
        }
    });

    stepState$ = this.stepStateSource.asObservable();

    private childNotificationSubject = new BehaviorSubject<ChildNotification | null>(null);
    childNotification$ = this.childNotificationSubject.asObservable();

    triggerChildNotification(message: string, className: string): void {
        this.childNotificationSubject.next({ message, className });
    }

    // Flags to ensure alerts are shown only once
    private otpWarningShown = false;
    private otpExpiredShown = false;
    private sessionWarningShown = false;
    private sessionExpiredShown = false;

    private otpWarningSubject = new BehaviorSubject<string | null>(null);
    private otpExpiredSubject = new BehaviorSubject<string | null>(null);
    private sessionWarningSubject = new BehaviorSubject<string | null>(null);
    private sessionExpiredSubject = new BehaviorSubject<string | null>(null);

    otpWarning$ = this.otpWarningSubject.asObservable();
    otpExpired$ = this.otpExpiredSubject.asObservable();
    sessionWarning$ = this.sessionWarningSubject.asObservable();
    sessionExpired$ = this.sessionExpiredSubject.asObservable();

    updateStatesForRoute(currentRoute: string): void {
        switch (currentRoute) {
            case 'email-verification':
                this.stepStateSource.next({
                    emailVerification: {
                        icon: this.ICONS.visited1,
                        textClass: this.STYLES.activeBold
                    },
                    personalInformation: {
                        icon: this.ICONS.unvisited2,
                        textClass: this.STYLES.inactive
                    },
                    accountConfirmation: {
                        icon: this.ICONS.unvisited3,
                        textClass: this.STYLES.inactive
                    }
                });
                break;

            case 'personal-information':
                this.stepStateSource.next({
                    emailVerification: {
                        icon: this.ICONS.done,
                        textClass: this.STYLES.completed
                    },
                    personalInformation: {
                        icon: this.ICONS.visited2,
                        textClass: this.STYLES.activeBold
                    },
                    accountConfirmation: {
                        icon: this.ICONS.unvisited3,
                        textClass: this.STYLES.inactive
                    }
                });
                break;

            case 'account-confirmation':
                this.stepStateSource.next({
                    emailVerification: {
                        icon: this.ICONS.done,
                        textClass: this.STYLES.completed
                    },
                    personalInformation: {
                        icon: this.ICONS.done,
                        textClass: this.STYLES.completed
                    },
                    accountConfirmation: {
                        icon: this.ICONS.done,
                        textClass: this.STYLES.completed
                    }
                });
                break;
        }
    }

    trackExpirations(): void {
        const otpExpiryStr = sessionStorage.getItem('registrationOtpExpiryTime');
        const sessionExpiryStr = sessionStorage.getItem('registrationFlowExpiresAt');
        const now = new Date();

        // OTP Expiration
        if (otpExpiryStr) {
            const otpExpiryDate = new Date(otpExpiryStr);
            const otpTimeLeft = otpExpiryDate.getTime() - now.getTime();
            if (otpTimeLeft > 0 && otpTimeLeft <= 60000 && !this.otpWarningShown) {
                this.otpWarningSubject.next('1 more minute to go before the OTP expires.');
                this.otpWarningShown = true;
            } else if (otpTimeLeft <= 0 && !this.otpExpiredShown) {
                this.otpExpiredSubject.next('Your OTP has expired.');
                this.otpExpiredShown = true;
            } else if (otpTimeLeft <= 0) {
                // Clear expired OTP
                sessionStorage.removeItem('registrationOtpExpiryTime');
            }
        }

        // Session Expiration
        if (sessionExpiryStr) {
            const sessionExpiryDate = new Date(sessionExpiryStr);
            const sessionTimeLeft = sessionExpiryDate.getTime() - now.getTime();
            if (sessionTimeLeft > 0 && sessionTimeLeft <= 60000 && !this.sessionWarningShown) {
                this.sessionWarningSubject.next('1 more minute till your session expires.');
                this.sessionWarningShown = true;
            } else if (sessionTimeLeft <= 0 && !this.sessionExpiredShown) {
                this.sessionExpiredSubject.next('Your session has expired.');
                this.sessionExpiredShown = true;
            } else if (sessionTimeLeft <= 0) {
                // Clear expired session
                sessionStorage.removeItem('registrationFlowExpiresAt');
            }
        }
    }

    markEmailAsVerified(): void {
        this.emailVerified = true;
    }

    isEmailVerified(): boolean {
        return this.emailVerified;
    }

    markPersonalInfoAsDone(): void {
        this.personalInfoCompletion = true;
    }

    isPersonalInfoDone(): boolean {
        return this.personalInfoCompletion;
    }

    resetFlags(): void {
        // Reset all alert flags to ensure proper re-triggering
        this.otpWarningShown = false;
        this.otpExpiredShown = false;
        this.sessionWarningShown = false;
        this.sessionExpiredShown = false;
        this.otpWarningSubject.next(null);
        this.otpExpiredSubject.next(null);
        this.sessionWarningSubject.next(null);
        this.sessionExpiredSubject.next(null);
        this.childNotificationSubject.next(null);
    }

    resetOtpState(): void {
        sessionStorage.removeItem('registrationOtpExpiryTime');
        this.otpWarningShown = false;
        this.otpExpiredShown = false;
        this.otpWarningSubject.next(null);
        this.otpExpiredSubject.next(null);
    }
}
This is how step - 1 the email verification component is handling the step tracker thing: 
import { Component, ElementRef, EventEmitter, HostListener, Output, QueryList, ViewChild, ViewChildren, ViewContainerRef } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { NotificationRef, NotificationService } from '@progress/kendo-angular-notification';
import { UserLoginService } from '../../services/user-login-service/user-login.service';
import { Router } from '@angular/router';
import { Subscription } from 'rxjs';
import { RegistrationFlowTokenService } from '../../services/registration-services/registration-token-service/registration-flow-token.service';
import { VerifyOtpService } from '../../services/registration-services/verify-otp-service/verify-otp.service';
import { ResendOtpService } from '../../services/registration-services/resend-otp-service/resend-otp.service';
import { environment } from '../../../environments/environment';
import { RegistrationStepTrackerService } from '../../services/helper-services/registration-step-tracker/registration-step-tracker.service';

@Component({
    selector: 'app-email-verification',
    templateUrl: './email-verification.component.html',
    styleUrl: './email-verification.component.css'
})
export class EmailVerificationComponent {
    @ViewChild('notificationContainer', { read: ViewContainerRef, static: true }) notificationContainer!: ViewContainerRef;
    @ViewChildren('otpInput') otpInputs!: QueryList<ElementRef>;
    emailForm: FormGroup;
    showValidationErrors: boolean = false;
    isEmailFilled: boolean = false;
    private backNavigationSubscription!: Subscription;
    dialogVisible = false;
    private lastNotificationRef: NotificationRef | null = null;
    showOtpSection: boolean = false;
    otpBoxes = ['', '', '', '', '', ''];
    isVerifyButtonDisabled: boolean = true;
    isPinVerified: boolean = false;
    pinCounter: number = 0;
    disableInputs: boolean = false;
    loaderVisible: boolean = false;
    verifyPinLoaderVisible: boolean = false;
    public loaderSize: 'small' | 'medium' = 'medium';
    private email: string = '';
    private otpCheckInterval!: any;
    private otpExpiredSubChild!: Subscription;

    constructor(private fb: FormBuilder,
        private userLoginService: UserLoginService,
        private router: Router,
        private verifyOtpService: VerifyOtpService,
        private registrationFlowTokenService: RegistrationFlowTokenService,
        private notificationService: NotificationService,
        private resendOtpService: ResendOtpService,
        private stepTrackerService: RegistrationStepTrackerService) {
        this.emailForm = this.fb.group(
            {
                email: [
                    '',
                    [
                        Validators.required,
                        Validators.email
                    ],
                ],
                confirmEmail: [{ value: '', disabled: true }, [Validators.required, Validators.email]],
            },
            {
                validators: this.emailsMatchValidator,
            }
        );
        const navigation = this.router.getCurrentNavigation();
    }

    @HostListener('window:resize')
    onResize() {
        this.updateLoaderSize();
    }

    ngOnInit(): void {
        this.backNavigationSubscription = this.userLoginService.backNavigation$
            .subscribe(shouldNavigateBack => {
                if (shouldNavigateBack) {
                    this.openDialog();
                    this.userLoginService.resetBackNavigation();
                }
            });

        this.updateLoaderSize();
        window.history.pushState(null, '', window.location.href);
        window.onpopstate = () => {
            window.history.pushState(null, '', window.location.href);
            this.openDialog();
        };
        this.otpExpiredSubChild = this.stepTrackerService.otpExpired$.subscribe(message => {
            if (message) {
                this.doMaxTriesPin(message);
            }
        });
        //this.stepTrackerService.triggerChildNotification('Doing a random notification for testing', 'custom-notification correct');
    }

    ngOnDestroy(): void {
        if (this.backNavigationSubscription) {
            this.backNavigationSubscription.unsubscribe();
        }
        window.onpopstate = null;
        if (this.otpExpiredSubChild) {
            this.otpExpiredSubChild.unsubscribe();
        }
    }

    private updateLoaderSize(): void {
        this.loaderSize = window.innerWidth < 425 ? 'small' : 'medium';
    }

    sanitizeInput(event: any): void {
        //const sanitizedValue = event.target.value.replace(/['";]/g, '');
        //event.target.value = sanitizedValue;
        const inputEl = event.target as HTMLInputElement;
        const forbiddenRegex = /[`!#$%\^{},|\[\]\\'()\?\/<>\;=]/g;

        const cleaned = inputEl.value.replace(forbiddenRegex, '');
        inputEl.value = cleaned;

        const controlName = inputEl.getAttribute('formControlName');
        if (controlName) {
            const ctrl = this.emailForm.get(controlName);
            if (ctrl) {
                ctrl.setValue(cleaned, { emitEvent: false });
            }
        }
    }

    emailsMatchValidator(group: FormGroup) {
        const email = group.get('email')?.value;
        const confirmEmail = group.get('confirmEmail')?.value;
        if (!confirmEmail) {
            return null;
        }
        return email === confirmEmail ? null : { emailsDoNotMatch: true };
    }

    onEmailInput(): void {
        const emailControl = this.emailForm.get('email');
        const confirmEmailControl = this.emailForm.get('confirmEmail');
        if (emailControl?.value) {
            confirmEmailControl?.enable();
        } else {
            confirmEmailControl?.disable();
            confirmEmailControl?.reset();
        }
    }

    onNext(): void {
        this.showValidationErrors = true;
        const enteredEmail = this.emailForm.get('email')?.value;
        if (this.emailForm.valid) {
            this.loaderVisible = true;
            this.registrationFlowTokenService.verifyEmail(enteredEmail).subscribe({
                next: (response) => {
                    const { message, token, expiresAt } = response;
                    if (token && expiresAt) {
                        sessionStorage.removeItem('registrationOtpExpiryTime');
                        sessionStorage.removeItem('registrationFlowExpiresAt');
                        this.registrationFlowTokenService.saveToken(token);
                        sessionStorage.setItem('registrationFlowExpiresAt', expiresAt);
                        this.setOtpExpiry();
                        this.triggerNotification(response?.message, 'custom-notification correct');
                        this.showOtpSection = true;
                    }
                },
                error: (error) => {
                    this.loaderVisible = false;
                    this.triggerNotification(error?.error?.message,'custom-notification incorrect')
                }
            });
        } else {
            console.log('Form is invalid:');
        }
    }

    setOtpExpiry(): void {
        const registrationOtpExpiryTime = new Date(Date.now() + 5 * 60 * 1000);
        sessionStorage.setItem('registrationOtpExpiryTime', registrationOtpExpiryTime.toISOString());
    }

    onCancel(): void {
        this.openDialog();
    }

    openDialog(): void {
        this.dialogVisible = true;
    }

    closeDialog(): void {
        this.dialogVisible = false;
    }

    onConfirm(): void {
        this.closeDialog();
        this.registrationFlowTokenService.clearToken();
        location.href = `${environment.homepage}`;
    } 

    onDecline(): void {
        this.closeDialog();
    }

    triggerNotification(notificationMessage: string, className: string): void {
        this.stepTrackerService.triggerChildNotification(notificationMessage, className);
    }

    onFocus(event: FocusEvent): void {
        const input = event.target as HTMLInputElement;
        input.placeholder = '';
        input.style.textAlign = 'center';
    }

    onBlur(event: FocusEvent): void {
        const input = event.target as HTMLInputElement;
        if (input.value === '') {
            input.placeholder = '--';
        }
    }

    movementFunction(event: KeyboardEvent, index: number): void {
        if (event.ctrlKey && event.key.toLowerCase() === 'v') {
            return;
        }
        const input = event.target as HTMLInputElement;
        if (event.key.match(/[0-9]/)) {
            input.value = event.key;
            const nextInput = this.otpInputs.toArray()[index + 1];
            if (nextInput) {
                nextInput.nativeElement.focus();
            }
        } else if (event.key === 'Backspace') {
            if (input.value !== '') {
                input.value = '';
                input.placeholder = '--';
            } else {
                const previousInput = this.otpInputs.toArray()[index - 1];
                if (previousInput) {
                    previousInput.nativeElement.focus();
                    previousInput.nativeElement.placeholder = '--';
                }
            }
        }
        event.preventDefault();
        this.validateInputs();
    }

    validateInputs(): void {
        if (this.disableInputs) {
            this.otpInputs.forEach(input => {
                input.nativeElement.disabled = true;
            });
        }
        this.isVerifyButtonDisabled = this.otpInputs.toArray().some(input => {
            const value = input.nativeElement.value.trim();
            return value === '' || isNaN(Number(value));
        });
    }

    verifyPin(): void {
        if (!this.disableInputs) {
            const enteredPin = this.otpInputs.toArray()
                .map(input => input.nativeElement.value)
                .join('');
            const enteredEmail = this.emailForm.get('email')?.value;
            this.verifyPinLoaderVisible = true;
            this.verifyOtpService.verifyPin(enteredEmail, enteredPin).subscribe({
                next: (response) => {
                    const successMessage = response.message || 'Pin Verified Successfully';
                    this.triggerNotification(successMessage, 'custom-notification correct');
                    this.isPinVerified = true;
                    this.stepTrackerService.markEmailAsVerified();
                    this.disableInputs = true;
                    this.verifyPinLoaderVisible = false;
                },
                error: (error) => {
                    const errorMessage = error.error?.message || 'My own error';
                    const finalErrorMessage = errorMessage.match("expired") ? true : false;
                    finalErrorMessage ? this.doMaxTriesPin(errorMessage)
                        : this.doIncorrectPin(errorMessage);
                    this.verifyPinLoaderVisible = false;
                }
            });
            this.resetPinInputs();
        }
    }

    onResendClick(event: Event): void {
        event.preventDefault(); // Prevents the default anchor navigation
        if (!this.isPinVerified) {
            this.resendPin();
            this.resetPinInputs();
        }
    }

    resetPinInputs(): void {
        this.otpInputs.forEach(input => {
            input.nativeElement.value = '';
            input.nativeElement.placeholder = '--';
        });
        this.isVerifyButtonDisabled = true;
    }

    resendPin(): void {
        const enteredEmail = this.emailForm.get('email')?.value;
        this.resendOtpService.triggerResend(enteredEmail).subscribe({
            next: (response) => {
                const successMessage = response.message || 'Pin Resent Successfully';
                this.disableInputs = false;
                this.triggerNotification(successMessage, 'custom-notification info');
                this.setOtpExpiry();
                this.stepTrackerService.resetFlags();
            },
            error: (error) => {
                const errorMessage = error.error?.message || 'My own error';
                this.triggerNotification(errorMessage, 'custom-notification incorrect');
            }
        });
    }

    onPrevious(): void {
        this.showOtpSection = false;
        this.registrationFlowTokenService.clearToken();
    }

    handleExpiredPin(message: string): void {
        this.triggerNotification(message, 'custom-notification info');
        this.disableInputs = true;
    }

    doIncorrectPin(message: string): void {
        this.triggerNotification(message, 'custom-notification incorrect');
    }

    doMaxTriesPin(message: string): void {
        this.triggerNotification(message, 'custom-notification incorrect');
        this.disableInputs = true;
    }

    onNextAgain(): void {
        if (this.lastNotificationRef) { this.lastNotificationRef.hide(); }
        const enteredEmail = this.emailForm.get('email')?.value;
        this.router.navigate(['/account-registration/personal-information'], { state: { email: enteredEmail } });
    }

    handleOtpPaste(event: ClipboardEvent): void {
        event.preventDefault();
        const pastedData = event.clipboardData?.getData('text') || '';
        const digits = pastedData.replace(/\D/g, '');
        const inputs = this.otpInputs.toArray();
        for (let i = 0; i < inputs.length; i++) {
            inputs[i].nativeElement.value = digits.charAt(i) || '';
        }
        this.validateInputs();
        if (inputs.length > 0) {
            inputs[inputs.length - 1].nativeElement.focus();
        }
    }

    handleConfirmEmailEnter(event: any): void {
        event.preventDefault();
        if (this.emailForm.valid) {
            this.onNext();
        }
    }

    onOtpEnter(): void {
        if (!this.isVerifyButtonDisabled) {
            this.verifyPin();
        }
    }
}
