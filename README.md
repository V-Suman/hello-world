Goal: To add checkboxes to each of the inputs next to their labels (Renewal Date, Paid Date, and Qualified Date) such that we enable the individual values. 
Rules: 
Currently as it stands.. when the user clicks the update notary appointment checkbox all the inputs (Renewal Date, Paid Date, and Qualified Date) get activated and then the 
user can sibsequently submit the input boxes with/without changing the dates for inputs (Renewal Date, Paid Date, and Qualified Date). If the user changed the input value of 
any of the inputs we console.log those. If the user did not change any we simply just console.log the already patched ones. THat is the current mechanism. And the moment the 
user checks the update notary appointment checkbox.. we enable the save button. 

But, I want to implement 3 checkboxes for each of the inputs (Renewal Date, Paid Date, and Qualified Date) next to their labels which will be in a disabled state initially. 
Such that.. When the user clicks the udate notary appointment checkbox We enable the checkboxes but, WE STILL DO NOT ENABLE THE SAVE button. 
But, instead ONLY IF the user clicks the respective checkbox then we enable the save button. Also, the user can click the update notary appointment chekbox and then click a 
respective checkbox.. and NOT edit the value.. we will have to consider whatever patched value we have as the console.log value (meaning final) value. 

Ask all clarifying questions before you get to implementation

HTML file: 
<div class="page-wrapper">
  <div class="heading-and-asterisk">
    <h2 class="notary-title">
      Update Notary Details – {{ temp }}
    </h2>
    <div class="required-indicator">
      <div class="asterisk">*</div>
      <div class="required-indicator-text"> – Required fields</div>
    </div>
  </div>

  <form [formGroup]="form" (ngSubmit)="onSubmit()" class="form-class" novalidate>
    <!-- ADD NOTES -->
    <div class="notes-section" formGroupName="addNotes">
      <div class="div-header-section">
        <input type="checkbox"
               kendoCheckBox
               formControlName="enabled"
               class="checkbox-override" />
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
          <kendo-formfield class="flex-item">
            <label kendoLabel for="renewalDate">
              Renewal Date <span class="text-danger asterisk">*</span>
            </label>
            <kendo-datepicker id="renewalDate"
                              formControlName="renewalDate"
                              [format]="'MM/dd/yyyy'"
                              placeholder="MM/DD/YYYY"
                              [min]="renewalMin"
                              [max]="renewalMax">
            </kendo-datepicker>
          </kendo-formfield>

          <kendo-formfield class="flex-item">
            <label kendoLabel for="paidDate">
              Paid Date <span class="text-danger asterisk">*</span>
            </label>
            <kendo-datepicker id="paidDate"
                              formControlName="paidDate"
                              [format]="'MM/dd/yyyy'"
                              placeholder="MM/DD/YYYY"
                              [min]="paidMin"
                              [max]="paidMax">
            </kendo-datepicker>
          </kendo-formfield>

          <kendo-formfield class="flex-item">
            <label kendoLabel for="qualifiedDate">
              Qualified Date <span class="text-danger asterisk">*</span>
            </label>
            <kendo-datepicker id="qualifiedDate"
                              formControlName="qualifiedDate"
                              [format]="'MM/dd/yyyy'"
                              placeholder="MM/DD/YYYY">
            </kendo-datepicker>
          </kendo-formfield>
        </div>
      </div>
    </div>

    <div class="valid-certificate-section" formGroupName="validCertificate">
      <div class="div-header-section">
        <input type="checkbox"
               kendoCheckBox
               formControlName="enabled"
               class="checkbox-override" />
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
               class="checkbox-override" />
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

        <!--<div class="form-column">
          <kendo-formfield class="flex-item">
            <label kendoLabel for="isResolved">Resolved?</label>
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
        </div>-->

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
              [disabled]="!hasAnySectionEnabled || form.invalid">
        Save
      </button>
    </div>
  </form>
</div>
import { Component, OnInit } from '@angular/core';
import { AbstractControl, FormBuilder, FormGroup, Validators } from '@angular/forms';
import { ActivatedRoute, ActivatedRouteSnapshot, Router } from '@angular/router';
import { ApprovalDate, RefGetterService } from '../../../services/helper-services/ref-get-service/ref-getter.service';
import { BuildEditObject } from '../../../models/update-profile-info-model/update-profile-info.model';

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
  public accountId: number = 0;
  today = this.stripTime(new Date());
  renewalMin = this.today;
  renewalMax = new Date(2100, 11, 31); // Dec 31, 2100
  paidMin = new Date(1899, 11, 31);    // Dec 31, 1899
  paidMax = this.today;

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
      (['renewalDate', 'paidDate', 'qualifiedDate'] as const).forEach(name => {
        const c = g.get(name)!;
        enabled ? c.enable({ emitEvent: false }) : c.disable({ emitEvent: false });
      });
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

    this.refGetter.getApprovalDates('U').subscribe({
      next: (res) => {
        this.approvalDates = res ?? [];
        const first = res?.[0]?.nextApprovalDate;          // "YYYY-MM-DD"
        const seed = this.parseAsLocalDate(first) ?? this.today;

        const renewalInit = this.clampDate(seed, this.renewalMin, this.renewalMax); // Today → 12/31/2100
        const paidInit = this.clampDate(seed, this.paidMin, this.paidMax);       // 12/31/1899 → Today
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
      || !!this.updateAppointmentEnabledCtrl.value
      || !!this.validCertificateEnabledCtrl.value
      || !!this.addComplaintEnabledCtrl.value;
  }

  private addMonths(date: Date, months: number): Date {
    const d0 = new Date(date);
    const d = new Date(date);
    d.setMonth(d.getMonth() + months);
    // handle month overflow (e.g., Jan 31 + 1 month -> Feb end)
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
    const payload: { notes?: string } = {};
    if (this.addNotesEnabledCtrl.value) {
      payload.notes = this.notesCtrl.value;
      console.log('SUBMIT payload:', payload);
    }

    if (this.updateAppointmentEnabledCtrl.value) {
      // Always read whatever is currently in the controls (updated or initial patched)
      const renewal = this.renewalCtrl.value as Date | null;
      const paid = this.paidCtrl.value as Date | null;
      const qualified = this.qualifiedCtrl.value as Date | null;

      // Always log all three (even if user changed only one)
      console.log({ applicationId: this.accountId, approvalDate: this.toSql(renewal) });
      console.log({ applicationId: this.accountId, payDate: this.toSql(paid) });
      console.log({ applicationId: this.accountId, qualifiedDate: this.toSql(qualified) });
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
