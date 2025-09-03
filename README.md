Goal: To integrate the interfaces available in my model.ts file with the data in the update-notary-details.component.ts file
such that.. when the save button is clicked.. the data that gets console.logged is exactly of the shape of the interfaces
present in the model file. 
MOre Details: Up until now.. we console.logged the data in any format we could.. going forward when the save button is 
clicked and data is console.logged I want the data to follow the interfaces present in the model file. 
1. For add notes section use the AddNotesRequest interface 
2. For Update notary appointment's renewal date.. use the RenewalDateRequest
3. For update notary appointment's paid date.. use the PaidDateRequest
4. For update notary appointment's qualified date.. use the QualifiedDateRequest
5. For valid certificate section use ValidCertificateRequest
6. For Add complaint section use AddComplaint interface. 

So when I click save and something gets console.logged it should follow the interface. 
Before you continue add all clarifying questions/doubts. 
update-notary-details.model.ts file:
export interface AddNotesRequest {
    accountId: number | null;
    notes: string;
}
export interface RenewalDateRequest {
    applicantId: number;
    approvalDate: string;
}
export interface PaidDateRequest {
    transactionStatusId: number;
    applicationId: number;
    transactionTypeId: number;
    amount: number;
    expeditedFee: number;
    payId: number;
    ccaId: number;
    paymentNum: string;
    payDate: string;
    comment: string;
    requestingHost: string;
    loggedInUserEmail: string;
}
export interface QualifiedDateRequest {
    applicationId: number;
    qualifiedDate: string;
}
export interface ValidCertificateRequest {
    accountId: number | null;
    certificateNumber: string;
    validationStartDate: string;
    validationEndDate: string;
}
export interface AddComplaint {
    accountId: number | null;
    dateOfComplaint: string;
    isRoncomplaint: boolean;
    complaintDetails: string;
    isResolved: boolean;
    resolutionDate: string;
    resolutionNotes: string;
}
update-notary-details.component.ts file:
import { Component, OnInit } from '@angular/core';
import { AbstractControl, FormBuilder, FormGroup, Validators } from '@angular/forms';
import { ActivatedRoute, ActivatedRouteSnapshot, Router } from '@angular/router';
import { ApprovalDate, RefGetterService } from '../../../services/helper-services/ref-get-service/ref-getter.service';
import { BuildEditObject } from '../../../models/update-profile-info-model/update-profile-info.model';
import { AddNotesRequest } from '../../../models/update-notary-details/update-notary-details.model';

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
;

    constructor(
        private router: Router,
        private route: ActivatedRoute,
        private fb: FormBuilder,
        private refGetter: RefGetterService
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

    ngOnInit(): void {
        this.temp = this.route.snapshot.paramMap.get('id');

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
        // Future-proof: extend this OR across other section checkboxes
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

    onSubmit(): void {

        if (!this.hasAnySectionEnabled) return;

        if (this.form.invalid) {
            this.form.markAllAsTouched();
            return;
        }

        // Build payload: only include sections that are enabled
        const payload: { notes?: string, accountId?: number | null } = {};
        if (this.addNotesEnabledCtrl.value) {
            payload.accountId = this.accountId;
            payload.notes = this.notesCtrl.value;
            console.log('SUBMIT payload:', payload);
        }


        if (this.updateAppointmentEnabledCtrl.value) {
            if (this.renewalSelectedCtrl.value) {
                console.log({ applicationId: this.accountId, approvalDate: this.toSql(this.renewalCtrl.value as Date | null) });
            }
            if (this.paidSelectedCtrl.value) {
                console.log({ applicationId: this.accountId, payDate: this.toSql(this.paidCtrl.value as Date | null) });
            }
            if (this.qualifiedSelectedCtrl.value) {
                console.log({ applicationId: this.accountId, qualifiedDate: this.toSql(this.qualifiedCtrl.value as Date | null) });
            }
        }

        if (this.validCertificateEnabledCtrl.value) {
            const cert = (this.validationCertificateCtrl.value ?? '').toString();
            const start = this.validationStartCtrl.value as Date | null;
            const end = this.validationEndCtrl.value as Date | null;

            console.log({ validationCertificate: cert, accountId: this.accountId });
            console.log({ validationStartDate: this.toSql(start), accountId: this.accountId });
            console.log({ validationEndDate: this.toSql(end), accountId: this.accountId });
        }

        if (this.addComplaintEnabledCtrl.value) {
            const dateOfComplaint = this.dateOfComplaintCtrl.value as Date | null;
            const isRoncomplaint = !!this.isRoncomplaintCtrl.value;
            const isResolved = !!this.isResolvedCtrl.value;
            const resolutionDate = this.resolutionDateCtrl.value as Date | null;

            const complaintDetails = this.sanitize(this.complaintDetailsCtrl.value as string);
            const resolutionNotes = this.sanitize(this.resolutionNotesCtrl.value as string);

            console.log({
                accountId: this.accountId,
                dateOfComplaint: this.toSql(dateOfComplaint),                 // Option A: UTC midnight
                isRoncomplaint,
                complaintDetails,
                isResolved,
                resolutionDate: isResolved ? this.toSql(resolutionDate) : null,
                resolutionNotes: isResolved ? resolutionNotes || null : null
            });
        }
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
