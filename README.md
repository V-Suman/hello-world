Goal: To generate the value of validation certificate number on checkbox click once and persist
that until the user navigates away from the page. 
Details: Until now.. whenever the page loads.. we pick up whatever is the user's accountId and patch that 
value in the validation certificate number input field. But, from now.. what needs to happen iss.. 
on the page initial load.. we will have to display a placeholder text "Certificate Number" in the 
validation certificate number input field and then once the user clicks the valid certificate checkbox..
that is when we need to make a call to the refGetter service's getCertificateNumber() method to fetch 
the certificate number from the backend. After fetching that value.. we need to patch it to the 
validation certificate number input field. Now from here on.. regardless of user unchecking the valid 
certificate checkbox or any action.. we don't make any further calls and keep the value of 
validation certificate number input field till the user navigates away from the update-notary-details page. 
Once the user navigates away.. we forget the value. The user can also submit the details too! 
Ask me any clarifying questions you have. 
ref-getter.service.ts file: 
import { Injectable } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { environment } from '../../../../environments/environment';
import { StateRef, SalutationRef, CityCountyRef } from '../../../models/ref-items-model/ref-items.model';

export interface ApprovalDate { nextApprovalDate: string; }
@Injectable({ providedIn: 'root' })
export class RefGetterService {
    private salutationsUrl = `${environment.apiurl}/api/Lookup/GetSalutations`;
    private statesUrl = `${environment.apiurl}/api/Lookup/GetStates`;
    private cityCountyUrl = `${environment.apiurl}/api/Lookup/GetCityTown`;
    private approvalDatesUrl = `${environment.apiurl}/api/InternalUser/GetNextApprovalDatesList`;
    private getCertificateUrl = `${environment.apiurl}/api/InternalUser/GetCertificateNumber`;


    constructor(private http: HttpClient) { }

    public getSalutations(): Observable<SalutationRef[]> {
        return this.http.get<SalutationRef[]>(this.salutationsUrl);
    }

    public getStates(): Observable<StateRef[]> {
        return this.http.get<StateRef[]>(this.statesUrl);
    }

    public getCityCounty(): Observable<CityCountyRef[]> {
        return this.http.get<CityCountyRef[]>(this.cityCountyUrl);
    }

    public getCertificateNumber(): Observable<String> {
        return this.http.get<String>(this.getCertificateUrl);
    }

    public getApprovalDates(transactionType: 'A' | 'U'): Observable<ApprovalDate[]> {
        if (transactionType !== 'A' && transactionType !== 'U') {
            return throwError(() => new Error('Invalid transactionType. Must be "A" or "U".'));
        }
        const params = new HttpParams().set('transactionType', transactionType);
        return this.http.get<ApprovalDate[]>(this.approvalDatesUrl, { params });
    }
}
update-notary-details.component.ts file: 
import { Component, OnInit } from '@angular/core';
import { AbstractControl, FormBuilder, FormGroup, Validators } from '@angular/forms';
import { ActivatedRoute, ActivatedRouteSnapshot, Router } from '@angular/router';
import { ApprovalDate, RefGetterService } from '../../../services/helper-services/ref-get-service/ref-getter.service';
import { BuildEditObject } from '../../../models/update-profile-info-model/update-profile-info.model';
import {
    AddNotesRequest,
    RenewalDateRequest,
    PaidDateRequest,
    QualifiedDateRequest,
    ValidCertificateRequest,
    AddComplaint
} from '../../../models/update-notary-details/update-notary-details.model';
import { UpdateNotaryDetailsService } from '../../../services/update-notary-details/update-notary-details.service';
import { forkJoin, of } from 'rxjs';
import { map, catchError, finalize } from 'rxjs/operators';

@Component({
    selector: 'app-update-notary-details',
    templateUrl: './update-notary-details.component.html',
    styleUrl: './update-notary-details.component.css'
})
export class UpdateNotaryDetailsComponent implements OnInit {
    temp: string | null = '0';
    form!: FormGroup;
    approvalDates: ApprovalDate[] = [];
    public displayData?: BuildEditObject;
    public accountId: number | null = 0;
    today = this.stripTime(new Date());
    renewalMin = this.today;
    renewalMax = new Date(2100, 11, 31); // Dec 31, 2100
    paidMin = new Date(1899, 11, 31);    // Dec 31, 1899
    paidMax = this.today;
    public applicantId: number = -1;
    submitting = false;

    constructor(
        private router: Router,
        private route: ActivatedRoute,
        private fb: FormBuilder,
        private refGetter: RefGetterService,
        private postNotaryDetails: UpdateNotaryDetailsService
    ) {
        // 1) Preferred: getCurrentNavigation (only on the initial nav)
        const nav = this.router.getCurrentNavigation();
        this.displayData = nav?.extras.state?.['buildEditObject'] as BuildEditObject | undefined;

        // 2) Fallback (and works after page reload):
        if (!this.displayData && history.state?.buildEditObject) {
            this.displayData = history.state.buildEditObject;
        }

        if (this.displayData) {
            this.accountId = this.displayData.personalInfoDetails.accountId;
        }

        console.log('Received buildEditObject:', this.accountId);
    }

    get isAccountless(): boolean {
        return (this.accountId ?? 0) === 0;
    }

    ngOnInit(): void {
        this.temp = this.route.snapshot.paramMap.get('id');
        const routeApplicantId = Number(this.temp);
        if (Number.isFinite(routeApplicantId) && routeApplicantId > 0) {
            this.applicantId = routeApplicantId;
        } else if ((this.displayData as any)?.personalInfoDetails?.applicantId > 0) {
            this.applicantId = (this.displayData as any).personalInfoDetails.applicantId;
        } else {
            console.error('Missing/invalid applicantId; Renewal Date submit will be blocked.');
        }


        // Parent form with nested section groups
        this.form = this.fb.group({
            addNotes: this.fb.group({
                enabled: [false], // checkbox state (not submitted)
                notes: [{ value: '', disabled: true }, [Validators.required, Validators.maxLength(5000)]]
            }),
            updateAppointment: this.fb.group({
                enabled: [false],
                renewalSelected: [{ value: false, disabled: true }],
                paidSelected: [{ value: false, disabled: true }],
                qualifiedSelected: [{ value: false, disabled: true }],

                renewalDate: [{ value: null, disabled: true }],
                paidDate: [{ value: null, disabled: true }],
                qualifiedDate: [{ value: null, disabled: true }]
            }),
            validCertificate: this.fb.group({
                enabled: [false],
                validationCertificate: [{ value: '', disabled: true },
                [Validators.maxLength(10), Validators.pattern(/^\d{0,10}$/)]
                ],
                validationStartDate: [{ value: null, disabled: true }],
                validationEndDate: [{ value: null, disabled: true }]
            }),
            addComplaint: this.fb.group({
                enabled: [false],

                dateOfComplaint: [{ value: null, disabled: true }, [Validators.required]],

                isRoncomplaint: [{ value: false, disabled: true }],

                complaintDetails: [{ value: '', disabled: true },
                [Validators.required, Validators.maxLength(200)]
                ],

                isResolved: [{ value: false, disabled: true }],

                // hidden until resolved; start at today (kept disabled until resolved)
                resolutionDate: [{ value: this.today, disabled: true }],

                resolutionNotes: [{ value: '', disabled: true }, [Validators.maxLength(200)]]
            }),
        });


        if (this.isAccountless) {
            (this.form.get('validCertificate') as FormGroup).disable({ emitEvent: false });
            (this.form.get('addComplaint') as FormGroup).disable({ emitEvent: false });
            // Ensure their "enabled" toggles stay false
            this.validCertificateEnabledCtrl.setValue(false, { emitEvent: false });
            this.addComplaintEnabledCtrl.setValue(false, { emitEvent: false });
        }

        // Wire the checkbox to the control's enable/disable & validators
        this.addNotesEnabledCtrl.valueChanges.subscribe((enabled: boolean) => {
            const notes = this.notesCtrl;
            if (enabled) {
                notes.enable({ emitEvent: false });
                notes.setValidators([Validators.required, Validators.maxLength(500)]);
            } else {
                notes.disable({ emitEvent: false });   // value preserved, excluded from validation
                notes.clearValidators();
            }
            notes.updateValueAndValidity({ emitEvent: false });
        });

        this.updateAppointmentEnabledCtrl.valueChanges.subscribe((enabled: boolean) => {
            const g = this.updateAppointmentGroup;

            // Enable/disable ONLY the perfield toggles
            (['renewalSelected', 'paidSelected', 'qualifiedSelected'] as const).forEach(name => {
                const c = g.get(name)!;
                enabled ? c.enable({ emitEvent: false }) : c.disable({ emitEvent: false });
            });

            // If master is turned off, forcedisable all datepickers (preserve values)
            if (!enabled) {
                (['renewalDate', 'paidDate', 'qualifiedDate'] as const).forEach(name => {
                    g.get(name)!.disable({ emitEvent: false });
                });
            }
        });

        this.validCertificateEnabledCtrl.valueChanges.subscribe((enabled: boolean) => {
            const g = this.validCertificateGroup;
            (['validationCertificate', 'validationStartDate', 'validationEndDate'] as const).forEach(name => {
                const c = g.get(name)!;
                enabled ? c.enable({ emitEvent: false }) : c.disable({ emitEvent: false });
            });
        });

        this.addComplaintEnabledCtrl.valueChanges.subscribe((enabled: boolean) => {
            const g = this.addComplaintGroup;
            // Toggle children
            (['dateOfComplaint', 'isRoncomplaint', 'complaintDetails', 'isResolved', 'resolutionNotes'] as const)
                .forEach(name => {
                    const c = g.get(name)!;
                    enabled ? c.enable({ emitEvent: false }) : c.disable({ emitEvent: false });
                });

            // resolutionDate stays disabled until isResolved = true
            const doc = this.dateOfComplaintCtrl;
            // If enabling and no date set, seed with today
            if (enabled && !doc.value) {
                doc.setValue(this.today, { emitEvent: false });
            }

            // Required validator for dateOfComplaint only when section is enabled
            if (enabled) {
                doc.setValidators([Validators.required]);
            } else {
                doc.clearValidators();
            }
            doc.updateValueAndValidity({ emitEvent: false });
        });

        this.isResolvedCtrl.valueChanges.subscribe((resolved: boolean) => {
            const rd = this.resolutionDateCtrl;
            const rn = this.resolutionNotesCtrl;

            if (resolved) {
                // enable + require both
                rd.enable({ emitEvent: false });
                rn.enable({ emitEvent: false });

                // seed date if empty
                if (!rd.value) rd.setValue(this.today, { emitEvent: false });

                rd.setValidators([Validators.required]);
                rn.setValidators([Validators.required, Validators.maxLength(200)]);
            } else {
                // optional + disabled (hidden in UI); keep values but not validated
                rd.disable({ emitEvent: false });
                rn.clearValidators();              // notes optional when not resolved
                rd.clearValidators();
            }

            rd.updateValueAndValidity({ emitEvent: false });
            rn.updateValueAndValidity({ emitEvent: false });
        });

        this.renewalSelectedCtrl.valueChanges.subscribe((on: boolean) => {
            on ? this.renewalCtrl.enable({ emitEvent: false })
                : this.renewalCtrl.disable({ emitEvent: false });
        });

        this.paidSelectedCtrl.valueChanges.subscribe((on: boolean) => {
            on ? this.paidCtrl.enable({ emitEvent: false })
                : this.paidCtrl.disable({ emitEvent: false });
        });

        this.qualifiedSelectedCtrl.valueChanges.subscribe((on: boolean) => {
            on ? this.qualifiedCtrl.enable({ emitEvent: false })
                : this.qualifiedCtrl.disable({ emitEvent: false });
        });

        this.refGetter.getApprovalDates('U').subscribe({
            next: (res) => {
                this.approvalDates = res ?? [];
                const first = res?.[0]?.nextApprovalDate;          // "YYYY-MM-DD"
                const seed = this.parseAsLocalDate(first) ?? this.today;

                const renewalInit = this.clampDate(seed, this.renewalMin, this.renewalMax); // Today  12/31/2100
                const paidInit = this.clampDate(seed, this.paidMin, this.paidMax);       // 12/31/1899  Today
                const qualifiedInit = seed;                                                    // no limits

                const startInit = seed;
                const endInit = this.addMonths(seed, 1);

                this.updateAppointmentGroup.patchValue(
                    { renewalDate: renewalInit, paidDate: paidInit, qualifiedDate: qualifiedInit },
                    { emitEvent: false }
                );

                this.validCertificateGroup.patchValue(
                    {
                        validationCertificate: String(this.accountId ?? ''),
                        validationStartDate: startInit,
                        validationEndDate: endInit
                    },
                    { emitEvent: false }
                );
            },
            error: (err) => {
                console.error('Failed to fetch approval dates', err);
            }
        });
    }

    get addComplaintGroup(): FormGroup {
        return this.form.get('addComplaint') as FormGroup;
    }
    get renewalSelectedCtrl(): AbstractControl {
        return this.updateAppointmentGroup.get('renewalSelected')!;
    }
    get paidSelectedCtrl(): AbstractControl {
        return this.updateAppointmentGroup.get('paidSelected')!;
    }
    get qualifiedSelectedCtrl(): AbstractControl {
        return this.updateAppointmentGroup.get('qualifiedSelected')!;
    }
    get addComplaintEnabledCtrl(): AbstractControl {
        return this.addComplaintGroup.get('enabled')!;
    }
    get dateOfComplaintCtrl(): AbstractControl {
        return this.addComplaintGroup.get('dateOfComplaint')!;
    }
    get isRoncomplaintCtrl(): AbstractControl {
        return this.addComplaintGroup.get('isRoncomplaint')!;
    }
    get complaintDetailsCtrl(): AbstractControl {
        return this.addComplaintGroup.get('complaintDetails')!;
    }
    get isResolvedCtrl(): AbstractControl {
        return this.addComplaintGroup.get('isResolved')!;
    }
    get resolutionDateCtrl(): AbstractControl {
        return this.addComplaintGroup.get('resolutionDate')!;
    }
    get resolutionNotesCtrl(): AbstractControl {
        return this.addComplaintGroup.get('resolutionNotes')!;
    }

    get validCertificateGroup(): FormGroup {
        return this.form.get('validCertificate') as FormGroup;
    }
    get validCertificateEnabledCtrl(): AbstractControl {
        return this.validCertificateGroup.get('enabled')!;
    }
    get validationCertificateCtrl(): AbstractControl {
        return this.validCertificateGroup.get('validationCertificate')!;
    }
    get validationStartCtrl(): AbstractControl {
        return this.validCertificateGroup.get('validationStartDate')!;
    }
    get validationEndCtrl(): AbstractControl {
        return this.validCertificateGroup.get('validationEndDate')!;
    }

    get updateAppointmentGroup(): FormGroup {
        return this.form.get('updateAppointment') as FormGroup;
    }
    get updateAppointmentEnabledCtrl(): AbstractControl {
        return this.updateAppointmentGroup.get('enabled')!;
    }
    get renewalCtrl(): AbstractControl {
        return this.updateAppointmentGroup.get('renewalDate')!;
    }
    get paidCtrl(): AbstractControl {
        return this.updateAppointmentGroup.get('paidDate')!;
    }
    get qualifiedCtrl(): AbstractControl {
        return this.updateAppointmentGroup.get('qualifiedDate')!;
    }

    private stripTime(d: Date): Date {
        const nd = new Date(d); nd.setHours(0, 0, 0, 0); return nd;
    }
    private parseAsLocalDate(s?: string): Date | null {
        if (!s) return null;
        const [y, m, d] = s.split('-').map(Number);
        return new Date(y, (m ?? 1) - 1, d ?? 1);
    }
    private toSql(date?: Date | null): string | null {
        if (!date) return null;
        const y = date.getFullYear(), m = date.getMonth(), d = date.getDate();
        return new Date(Date.UTC(y, m, d, 0, 0, 0)).toISOString();
    }

    private clampDate(date: Date | null, min: Date, max: Date): Date {
        const src = date ? new Date(date) : new Date();
        src.setHours(0, 0, 0, 0);
        const lo = new Date(min); lo.setHours(0, 0, 0, 0);
        const hi = new Date(max); hi.setHours(0, 0, 0, 0);
        const t = Math.min(Math.max(src.getTime(), lo.getTime()), hi.getTime());
        return new Date(t);
    }

    // ---- convenience getters ----
    get addNotesGroup(): FormGroup {
        return this.form.get('addNotes') as FormGroup;
    }
    get addNotesEnabledCtrl(): AbstractControl {
        return this.addNotesGroup.get('enabled')!;
    }
    get notesCtrl(): AbstractControl {
        return this.addNotesGroup.get('notes')!;
    }
    get hasAnySectionEnabled(): boolean {
        if (this.isAccountless) {
            return !!this.addNotesEnabledCtrl.value || (
                !!this.updateAppointmentEnabledCtrl.value && (
                    !!this.renewalSelectedCtrl.value ||
                    !!this.paidSelectedCtrl.value ||
                    !!this.qualifiedSelectedCtrl.value
                )
            );
        }
        return !!this.addNotesEnabledCtrl.value
            || !!this.validCertificateEnabledCtrl.value
            || !!this.addComplaintEnabledCtrl.value
            || (this.updateAppointmentEnabledCtrl.value && (
                !!this.renewalSelectedCtrl.value
                || !!this.paidSelectedCtrl.value
                || !!this.qualifiedSelectedCtrl.value
            ));
    }

    private addMonths(date: Date, months: number): Date {
        const d0 = new Date(date);
        const d = new Date(date);
        d.setMonth(d.getMonth() + months);
        // handle month overflow (e.g., Jan 31 + 1 month  Feb end)
        if (d.getDate() !== d0.getDate()) d.setDate(0);
        d.setHours(0, 0, 0, 0);
        return d;
    }

    private sanitize(text: string | null | undefined): string {
        if (!text) return '';
        // strip control chars, angle/brace/semicolon brackets, collapse whitespace, trim
        return text
            .replace(/[\x00-\x1F\x7F]/g, '')
            .replace(/[<>{}\[\];]/g, '')
            .replace(/\s+/g, ' ')
            .trim();
    }

    digitsOnly(e: KeyboardEvent) {
        const allowed = ['Backspace', 'Tab', 'ArrowLeft', 'ArrowRight', 'Delete', 'Home', 'End'];
        if (allowed.includes(e.key)) return;
        if (!/^\d$/.test(e.key)) e.preventDefault();
    }

    private toSqlOrToday(date: Date | null): string {
        return this.toSql(date) ?? this.toSql(this.today)!;
    }

    onSubmit(): void {
        if (!this.hasAnySectionEnabled) return;

        if (this.form.invalid) {
            this.form.markAllAsTouched();
            return;
        }

        // Build all selected calls
        type SectionResult = { label: string; ok: boolean };
        const calls: Array<import('rxjs').Observable<SectionResult>> = [];

        if (this.addNotesEnabledCtrl.value) {
            const req: AddNotesRequest = {
                applicantId: this.applicantId,   // send 0 when missing
                notes: (this.notesCtrl.value ?? '').toString()
            };
            console.log(req);
            calls.push(
                this.postNotaryDetails.addNotes(req).pipe(
                    map(ok => ({ label: 'Add Notes', ok: ok === true })),
                    catchError(() => of({ label: 'Add Notes', ok: false }))
                )
            );
        }

        if (this.updateAppointmentEnabledCtrl.value) {
            // Renewal
            if (this.renewalSelectedCtrl.value) {
                if (!(this.applicantId > 0)) {
                    alert('Missing applicantId for Renewal Date. Please navigate with a valid applicant id and try again.');
                    return; // block whole submit as requested
                }
                const req: RenewalDateRequest = {
                    applicantId: this.applicantId, // never 0
                    approvalDate: this.toYmdOrToday(this.renewalCtrl.value as Date | null)
                };
                console.log(req);
                calls.push(
                    this.postNotaryDetails.updateRenewalDate(req).pipe(
                        map(ok => ({ label: 'Renewal Date', ok: ok === true })),
                        catchError(() => of({ label: 'Renewal Date', ok: false }))
                    )
                );
            }

            // Paid
            if (this.paidSelectedCtrl.value) {
                const req: PaidDateRequest = {
                    applicantId: this.applicantId,
                    paidDate: this.toSqlOrToday(this.paidCtrl.value as Date | null),
                };
                console.log(req);
                calls.push(
                    this.postNotaryDetails.updatePaidDate(req).pipe(
                        map(ok => ({ label: 'Paid Date', ok: ok === true })),
                        catchError(() => of({ label: 'Paid Date', ok: false }))
                    )
                );
            }

            // Qualified
            if (this.qualifiedSelectedCtrl.value) {
                const req: QualifiedDateRequest = {
                    applicantId: this.applicantId,
                    qualifiedDate: this.toSqlOrToday(this.qualifiedCtrl.value as Date | null)
                };
                console.log(req);
                calls.push(
                    this.postNotaryDetails.updateQualifiedDate(req).pipe(
                        map(ok => ({ label: 'Qualified Date', ok: ok === true })),
                        catchError(() => of({ label: 'Qualified Date', ok: false }))
                    )
                );
            }
        }

        if (!this.isAccountless && this.validCertificateEnabledCtrl.value) {
            const req: ValidCertificateRequest = {
                accountId: this.accountId ?? 0,
                certificateNumber: (this.validationCertificateCtrl.value ?? '').toString(),
                validationStartDate: this.toYmdOrToday(this.validationStartCtrl.value as Date | null),
                validationEndDate: this.toYmdOrToday(this.validationEndCtrl.value as Date | null)
            };
            console.log(req);
            calls.push(
                this.postNotaryDetails.updateValidationCertificate(req).pipe(
                    map(ok => ({ label: 'Valid Certificate', ok: ok === true })),
                    catchError(() => of({ label: 'Valid Certificate', ok: false }))
                )
            );
        }

        if (!this.isAccountless && this.addComplaintEnabledCtrl.value) {
            const isResolved = !!this.isResolvedCtrl.value;
            const req: AddComplaint = {
                accountId: this.accountId ?? 0,
                dateOfComplaint: this.toSqlOrToday(this.dateOfComplaintCtrl.value as Date | null),
                isRoncomplaint: !!this.isRoncomplaintCtrl.value,
                complaintDetails: this.sanitize(this.complaintDetailsCtrl.value as string),
                isResolved,
                resolutionDate: isResolved ? this.toSqlOrToday(this.resolutionDateCtrl.value as Date | null) : null,
                resolutionNotes: isResolved ? (this.sanitize(this.resolutionNotesCtrl.value as string) || null) : null
            };
            console.log(req);
            calls.push(
                this.postNotaryDetails.addComplaint(req).pipe(
                    map(ok => ({ label: 'Add Complaint', ok: ok === true })),
                    catchError(() => of({ label: 'Add Complaint', ok: false }))
                )
            );
        }

        if (calls.length === 0) return;

        this.submitting = true;
        forkJoin(calls).pipe(finalize(() => (this.submitting = false))).subscribe(results => {
            const failed = results.filter(r => !r.ok).map(r => r.label);
            if (failed.length === 0) {
                
                if (this.temp != null) {
                    this.router.navigate(['/notary-profile', this.temp]);
                } else {
                    console.error('No applicantId available to navigate back.');
                }
                return;
            }

            if (failed.length === 1) {
                alert(`There is an error in the ${failed[0]} section. Could not save data.`);
            } else {
                alert(`There are errors in: ${failed.join(', ')} sections. Could not save data.`);
            }
        });
    }

    private toYmd(date: Date): string {
        const y = date.getFullYear();
        const m = String(date.getMonth() + 1).padStart(2, '0');
        const d = String(date.getDate()).padStart(2, '0');
        return `${y}-${m}-${d}`;
    }

    private toYmdOrToday(date: Date | null): string {
        return this.toYmd(date ?? this.today);
    }

    onCancel(evt?: Event): void {
        evt?.preventDefault();
        evt?.stopPropagation();
        if (this.temp != null) {
            this.router.navigate(['/notary-profile', this.temp]);
        } else {
            console.error('No applicantId available to navigate back.');
        }
    }
}
update-notary-details.component.html file: 
<div class="page-wrapper">
    <div class="heading-and-asterisk">
        <div class="title-and-tooltip">
            <h2 class="notary-title">
                Update Notary Details - {{ temp }}
            </h2>
            <div *ngIf="displayData?.personalInfoDetails?.accountId == 0" kendoTooltip position="right" class="tooltip-wrapper">
                <button kendoButton class="village-hyperlink" title="Some fields are unavailable to edit due to lack of a notary account">
                    ?
                </button>
            </div>
        </div>
        <div class="required-indicator">
            <div class="asterisk">*</div>
            <div class="required-indicator-text"> - Required fields</div>
        </div>
    </div>

    <form [formGroup]="form" (ngSubmit)="onSubmit()" class="form-class" novalidate>
        <!-- ADD NOTES -->
        <div class="notes-section" formGroupName="addNotes">
            <div class="div-header-section">
                <input type="checkbox"
                       kendoCheckBox
                       formControlName="enabled"
                       class="checkbox-override"
                       />
                <div class="checkbox-header-level">Add Notes</div>
            </div>

            <div class="div-content-section">
                <kendo-formfield class="full-width">
                    <label kendoLabel for="notes">
                        Notes <span class="text-danger asterisk padding-exception">*</span>
                    </label>
                    <!-- Disabled until checkbox is checked; value is preserved -->
                    <textarea kendoTextArea
                              id="notes"
                              formControlName="notes"
                              rows="6"
                              maxlength="5000"></textarea>
                    <kendo-formerror *ngIf="notesCtrl?.touched && notesCtrl?.errors?.['required']">
                        Notes are required.
                    </kendo-formerror>
                    <kendo-formerror *ngIf="notesCtrl?.touched && notesCtrl?.errors?.['maxlength']">
                        Max 500 characters.
                    </kendo-formerror>
                </kendo-formfield>
            </div>
        </div>
        <div class="appointment-section" formGroupName="updateAppointment">
            <div class="div-header-section">
                <input type="checkbox"
                       kendoCheckBox
                       formControlName="enabled"
                       class="checkbox-override" />
                <div class="checkbox-header-level">Update Notary Appointment</div>
            </div>

            <div class="div-content-section">
                <div class="form-column">
                    <div class="flex-item">
                        <div class="label-with-toggle">
                            <input type="checkbox" kendoCheckBox formControlName="renewalSelected" class="mini-checkbox" />
                            <label kendoLabel for="renewalDate">Renewal Date <span class="text-danger asterisk">*</span></label>
                        </div>
                        <kendo-formfield>
                            <kendo-datepicker id="renewalDate" formControlName="renewalDate" [format]="'MM/dd/yyyy'" placeholder="MM/DD/YYYY" [min]="renewalMin" [max]="renewalMax">
                            </kendo-datepicker>
                        </kendo-formfield>
                    </div>

                    <div class="flex-item">
                        <div class="label-with-toggle">
                            <input type="checkbox" kendoCheckBox formControlName="paidSelected" class="mini-checkbox" />
                            <label kendoLabel for="paidDate">Paid Date <span class="text-danger asterisk">*</span></label>
                        </div>
                        <kendo-formfield>
                            <kendo-datepicker id="paidDate" formControlName="paidDate" [format]="'MM/dd/yyyy'" placeholder="MM/DD/YYYY" [min]="paidMin" [max]="paidMax">
                            </kendo-datepicker>
                        </kendo-formfield>
                    </div>

                    <div class="flex-item">
                        <div class="label-with-toggle">
                            <input type="checkbox" kendoCheckBox formControlName="qualifiedSelected" class="mini-checkbox" />
                            <label kendoLabel for="qualifiedDate">Qualified Date <span class="text-danger asterisk">*</span></label>
                        </div>
                        <kendo-formfield>
                            <kendo-datepicker id="qualifiedDate" formControlName="qualifiedDate" [format]="'MM/dd/yyyy'" placeholder="MM/DD/YYYY">
                            </kendo-datepicker>
                        </kendo-formfield>
                    </div>
                </div>
            </div>
        </div>

        <div class="valid-certificate-section" formGroupName="validCertificate">
            <div class="div-header-section">
                <input type="checkbox"
                       kendoCheckBox
                       formControlName="enabled"
                       class="checkbox-override" 
                       [disabled]="isAccountless"/>
                <div class="checkbox-header-level">Valid Certificate</div>
            </div>

            <div class="div-content-section">
                <div class="form-column">
                    <kendo-formfield class="flex-item">
                        <label kendoLabel for="validationCertificate">
                            Validation Certificate #
                        </label>
                        <input kendoTextBox
                               id="validationCertificate"
                               formControlName="validationCertificate"
                               maxlength="10"
                               inputmode="numeric"
                               pattern="[0-9]*"
                               (keypress)="digitsOnly($event)" />
                        <kendo-formerror *ngIf="validationCertificateCtrl?.touched && validationCertificateCtrl?.errors?.['maxlength']">
                            Max 10 digits.
                        </kendo-formerror>
                        <kendo-formerror *ngIf="validationCertificateCtrl?.touched && validationCertificateCtrl?.errors?.['pattern']">
                            Numbers only.
                        </kendo-formerror>
                    </kendo-formfield>

                    <kendo-formfield class="flex-item">
                        <label kendoLabel for="validationStartDate">
                            Validation Start Date <span class="text-danger asterisk">*</span>
                        </label>
                        <kendo-datepicker id="validationStartDate"
                                          formControlName="validationStartDate"
                                          [format]="'MM/dd/yyyy'"
                                          placeholder="MM/DD/YYYY">
                        </kendo-datepicker>
                    </kendo-formfield>

                    <kendo-formfield class="flex-item">
                        <label kendoLabel for="validationEndDate">
                            Validation End Date <span class="text-danger asterisk">*</span>
                        </label>
                        <kendo-datepicker id="validationEndDate"
                                          formControlName="validationEndDate"
                                          [format]="'MM/dd/yyyy'"
                                          placeholder="MM/DD/YYYY">
                        </kendo-datepicker>
                    </kendo-formfield>
                </div>
            </div>
        </div>
        <div class="add-complaint-section" formGroupName="addComplaint">
            <div class="div-header-section">
                <input type="checkbox"
                       kendoCheckBox
                       formControlName="enabled"
                       class="checkbox-override" 
                       [disabled]="isAccountless"/>
                <div class="checkbox-header-level">Add Complaint</div>
            </div>

            <div class="div-content-section">
                <div class="form-column row-one">
                    <!-- Date of Complaint -->
                    <kendo-formfield class="flex-item">
                        <label kendoLabel for="dateOfComplaint">
                            Date of Complaint <span class="text-danger asterisk">*</span>
                        </label>
                        <kendo-datepicker id="dateOfComplaint"
                                          formControlName="dateOfComplaint"
                                          [format]="'MM/dd/yyyy'"
                                          placeholder="MM/DD/YYYY">
                        </kendo-datepicker>
                        <kendo-formerror *ngIf="dateOfComplaintCtrl?.touched && dateOfComplaintCtrl?.errors?.['required']">
                            Date of complaint is required.
                        </kendo-formerror>
                    </kendo-formfield>

                    <!-- Is RON Complaint -->
                    <kendo-formfield class="flex-item checkbox-option">
                        <input type="checkbox"
                               kendoCheckBox
                               id="isRoncomplaint"
                               formControlName="isRoncomplaint" />
                        <label kendoLabel for="isRoncomplaint" id="is-ron-complaint">Remote Online Complaint?</label>
                    </kendo-formfield>
                </div>

                <div class="form-column">
                    <!-- Complaint Details -->
                    <kendo-formfield class="full-width">
                        <label kendoLabel for="complaintDetails">
                            Complaint Details <span class="text-danger asterisk">*</span>
                        </label>
                        <textarea kendoTextArea
                                  id="complaintDetails"
                                  formControlName="complaintDetails"
                                  rows="4"
                                  maxlength="200"></textarea>
                        <kendo-formerror *ngIf="complaintDetailsCtrl?.touched && complaintDetailsCtrl?.errors?.['required']">
                            Complaint details are required.
                        </kendo-formerror>
                        <kendo-formerror *ngIf="complaintDetailsCtrl?.touched && complaintDetailsCtrl?.errors?.['maxlength']">
                            Max 200 characters.
                        </kendo-formerror>
                    </kendo-formfield>
                </div>

                <div class="form-column">
                    <!-- Resolution Notes -->
                    <kendo-formfield class="full-width resolution-override height-override">
                        <label kendoLabel *ngIf="isResolvedCtrl.value" for="resolutionNotes">
                            Resolution Notes <span class="text-danger asterisk">*</span>
                        </label>
                        <label kendoLabel *ngIf="!isResolvedCtrl.value" for="resolutionNotes">
                            Resolution Notes <span class="text-normal asterisk">*</span>
                        </label>
                        <textarea kendoTextArea
                                  id="resolutionNotes"
                                  formControlName="resolutionNotes"
                                  rows="4"
                                  maxlength="200"></textarea>
                        <kendo-formerror *ngIf="resolutionNotesCtrl?.touched && resolutionNotesCtrl?.errors?.['required']">
                            Resolution notes are required when resolved.
                        </kendo-formerror>
                        <kendo-formerror *ngIf="resolutionNotesCtrl?.touched && resolutionNotesCtrl?.errors?.['maxlength']">
                            Max 200 characters.
                        </kendo-formerror>
                    </kendo-formfield>
                    <div class="resolved-and-date">
                        <kendo-formfield class="flex-item checkbox-option">
                            <label kendoLabel for="isResolved" id="is-ron-complaint">Resolved?</label>
                            <input type="checkbox"
                                   kendoCheckBox
                                   id="isResolved"
                                   formControlName="isResolved" />
                        </kendo-formfield>

                        <kendo-formfield class="flex-item" *ngIf="isResolvedCtrl.value">
                            <label kendoLabel for="resolutionDate">
                                Resolution Date <span class="text-danger asterisk">*</span>
                            </label>
                            <kendo-datepicker id="resolutionDate"
                                              formControlName="resolutionDate"
                                              [format]="'MM/dd/yyyy'"
                                              placeholder="MM/DD/YYYY">
                            </kendo-datepicker>
                            <kendo-formerror *ngIf="resolutionDateCtrl?.touched && resolutionDateCtrl?.errors?.['required']">
                                Resolution date is required when resolved.
                            </kendo-formerror>
                        </kendo-formfield>
                    </div>
                </div>
            </div>
        </div>

        <div class="buttons-row">
            <button kendoButton
                    themeColor="primary"
                    type="button"
                    class="custom-button-alt"
                    (click)="onCancel($event)">
                Cancel
            </button>
            <button kendoButton
                    class="search-button"
                    themeColor="primary"
                    type="submit"
                    [disabled]="submitting || !hasAnySectionEnabled || form.invalid">
                Save
            </button>
        </div>
    </form>
</div>
