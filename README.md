I am picking up this component to work after a while. I will paste the code for ts and html file here. And then I will ask you some
questions tell me the answers for it. 
html file: 
<div class="page-wrapper">
  <div class="heading-and-asterisk">
    <h2 class="notary-title">
      Update Profile Information –
      {{ displayData?.personalInfoDetails?.firstName }} {{ displayData?.personalInfoDetails?.lastName }}
      ({{ displayData?.personalInfoDetails?.notaryIdentifier }})
    </h2>
    <div class="required-indicator">
      <div class="asterisk">*</div>
      <div class="required-indicator-text"> – Required fields</div>
    </div>
  </div>

  <form [formGroup]="personalForm">
    <!-- PERSONAL INFO -->
    <div class="personal-information-section">
      <div class="div-header-section">
        <input type="checkbox"
               kendoCheckBox
               (change)="togglePersonalInfo($event)"
               [checked]="updatePersonalChecked"
               class="checkbox-override" />
        <div class="checkbox-header-level">
          Update Personal Information
        </div>
      </div>
      <div class="div-content-section">
        <!-- Row 1 -->
        <div class="form-row">
          <kendo-formfield class="flex-item">
            <label kendoLabel for="salutation">
              Salutation <sup class="text-danger asterisk">*</sup>
            </label>
            <kendo-dropdownlist id="salutation"
                                formControlName="salutation"
                                [data]="salutationOptions"
                                [valuePrimitive]="true"
                                placeholder="Select...">
            </kendo-dropdownlist>
          </kendo-formfield>

          <kendo-formfield class="flex-item">
            <label kendoLabel for="firstName">
              First Name <sup class="text-danger asterisk">*</sup>
            </label>
            <input kendoTextBox id="firstName" formControlName="firstName" />
          </kendo-formfield>

          <kendo-formfield class="flex-item">
            <label kendoLabel for="middleName">
              Middle Name <sup class="text-danger disable-super">*</sup>
            </label>
            <input kendoTextBox id="middleName" formControlName="middleName" />
          </kendo-formfield>

          <kendo-formfield class="flex-item">
            <label kendoLabel for="lastName">
              Last Name <sup class="text-danger asterisk">*</sup>
            </label>
            <input kendoTextBox id="lastName" formControlName="lastName" />
          </kendo-formfield>

          <kendo-formfield class="flex-item">
            <label kendoLabel for="suffix">
              Suffix <sup class="text-danger disable-super">*</sup>
            </label>
            <input kendoTextBox id="suffix" formControlName="suffix" />
          </kendo-formfield>
        </div>

        <!-- Row 2 -->
        <div class="form-row">
          <kendo-formfield class="flex-item">
            <label kendoLabel for="dateOfBirth">
              Date of Birth <sup class="text-danger asterisk">*</sup>
            </label>
            <kendo-datepicker id="dateOfBirth"
                              formControlName="dateOfBirth"
                              [format]="'MM/dd/yyyy'"
                              placeholder="MM/DD/YYYY">
            </kendo-datepicker>
          </kendo-formfield>

          <kendo-formfield class="flex-item">
            <label kendoLabel for="dateOfDeath">
              Date of Death <sup class="text-danger disable-super">*</sup>
            </label>
            <kendo-datepicker id="dateOfDeath"
                              formControlName="dateOfDeath"
                              [format]="'MM/dd/yyyy'"
                              placeholder="MM/DD/YYYY">
            </kendo-datepicker>
          </kendo-formfield>

          <kendo-formfield class="flex-item">
            <label kendoLabel for="dateOfResignation">
              Date of Resignation <sup class="text-danger disable-super">*</sup>
            </label>
            <kendo-datepicker id="dateOfResignation"
                              formControlName="dateOfResignation"
                              [format]="'MM/dd/yyyy'"
                              placeholder="MM/DD/YYYY">
            </kendo-datepicker>
          </kendo-formfield>
        </div>

        <!-- Row 3 -->
        <div class="form-row">
          <kendo-formfield class="flex-item">
            <label kendoLabel for="email">
              Email Address <sup class="text-danger disable-super">*</sup>
            </label>
            <input kendoTextBox id="email" formControlName="email" />
          </kendo-formfield>

          <!-- Primary Phone -->
          <div class="flex-item">
            <label class="group-label">
              Primary Phone <sup class="text-danger asterisk">*</sup>
            </label>
            <div class="phone-group">
              <kendo-formfield>
                <input kendoTextBox
                       id="primaryPhone1"
                       formControlName="primaryPhone1"
                       maxlength="3"
                       class="three-size" />
              </kendo-formfield>
              <span class="dash">-</span>
              <kendo-formfield>
                <input kendoTextBox
                       id="primaryPhone2"
                       formControlName="primaryPhone2"
                       maxlength="3"
                       class="three-size" />
              </kendo-formfield>
              <span class="dash">-</span>
              <kendo-formfield>
                <input kendoTextBox
                       id="primaryPhone3"
                       formControlName="primaryPhone3"
                       maxlength="4"
                       class="four-size" />
              </kendo-formfield>
            </div>
          </div>

          <!-- Secondary Phone -->
          <div class="flex-item">
            <label class="group-label">
              Secondary Phone <sup class="text-danger disable-super">*</sup>
            </label>
            <div class="phone-group">
              <kendo-formfield>
                <input kendoTextBox
                       id="secondaryPhone1"
                       formControlName="secondaryPhone1"
                       maxlength="3"
                       class="three-size" />
              </kendo-formfield>
              <span class="dash">-</span>
              <kendo-formfield>
                <input kendoTextBox
                       id="secondaryPhone2"
                       formControlName="secondaryPhone2"
                       maxlength="3"
                       class="three-size" />
              </kendo-formfield>
              <span class="dash">-</span>
              <kendo-formfield>
                <input kendoTextBox
                       id="secondaryPhone3"
                       formControlName="secondaryPhone3"
                       maxlength="4"
                       class="four-size" />
              </kendo-formfield>
            </div>
          </div>
        </div>

        <!-- Row 4: Radio -->
        <div class="form-row">
            <label kendoLabel class="flex-item radio-modifier">
                Type of change <sup class="text-danger asterisk">*</sup>
            </label>
            <kendo-formfield class="flex-item radio-fields-modifier">
                <kendo-radiobutton formControlName="changeType"
                                   name="changeType"
                                   id="changeTypeOfficial"
                                   value="Official">
                </kendo-radiobutton>
                <label for="changeTypeOfficial">Official</label>
            </kendo-formfield>
            <kendo-formfield class="flex-item radio-fields-modifier">
                <kendo-radiobutton formControlName="changeType"
                                   name="changeType"
                                   id="changeTypeCorrection"
                                   value="Correction">
                </kendo-radiobutton>
                <label for="changeTypeCorrection">Correction</label>
            </kendo-formfield>
            <div class="error-message">
                <span *ngIf="personalForm && personalForm.get('changeType')?.hasError('required') && showValidationErrors">
                    You must select a type of change.
                </span>
            </div>
        </div>
      </div>
    </div>
  </form>

  <!-- ADDRESS INFO FORM -->
  <form [formGroup]="addressForm">
    <div class="address-information-section">
      <div class="div-header-section">
        <input type="checkbox"
               kendoCheckBox
               (change)="toggleAddressInfo($event)"
               [checked]="updateAddressChecked"
               class="checkbox-override" />
        <div class="checkbox-header-level">
          Update Address Information
        </div>
      </div>
      <div class="div-content-section address-div">
        <!-- Row 1: Preferred Radio -->
        <div class="form-row address-preference-row">
          <span class="flex-item exception-flex-item">
            Preferred address for communication? <sup class="question-asterisk">*</sup>
          </span>
          <input type="radio" formControlName="preferredAddress" value="Residence" />
          <label class="flex-item">Residence</label>

          <input type="radio" formControlName="preferredAddress" value="Business" />
          <label class="flex-item">Business</label>
        </div>

        <!-- Row 2: Residence Address -->
        <div class="form-row">
          <div class="flex-item" formGroupName="residenceAddress">
            <div class="heading-and-error">
              <div class="section-heading">
                Residence Address (PO boxes not accepted)
              </div>
              <span class="error-message error-message-with-top-marg" *ngIf="submittedAddress && preferred === 'Business' && addressForm.get('residenceAddress')?.invalid">
                Please complete all the fields or none.
              </span>
            </div>

            <!-- Street -->
            <div class="label-and-error-message">
              <label class="label-and-asterisk" kendoLabel>Street Address <sup class="text-danger" [class.disable-super]="preferred==='Business'">*</sup></label>
              <div class="error-message">
                <span *ngIf="submittedAddress && addressForm.get('residenceAddress')?.hasError('noStreet')">
                  Street number and Address Line are required.
                </span>
              </div>
            </div>
            <div class="form-row-no-wrap">
              <input kendoTextBox formControlName="street1" id="street-number" placeholder="Street No." />
              <input kendoTextBox formControlName="street2" id="address-line" placeholder="Address Line" />
              <input kendoTextBox formControlName="street3" id="address-suffix" placeholder="Suffix" />
            </div>
            <div class="form-row">
              <input kendoTextBox formControlName="street4" placeholder="Address Line 2" />
            </div>

            <!-- State / Zip -->
            <div class="form-row state-and-zip">
              <kendo-formfield class="flex-item">
                <label class="label-and-asterisk" kendoLabel>
                  State
                  <sup class="text-danger" [class.disable-super]="preferred==='Business'">
                    *
                  </sup>
                  <button kendoPopoverAnchor
                          [popover]="pop"
                          showOn="hover"
                          class="village-hyperlink"
                          kendoButton>
                    ?
                  </button>

                  <kendo-popover #pop title="Need help?" [width]="400">
                    <!-- this template will override the `body` property -->
                    <ng-template kendoPopoverBodyTemplate>
                      <p>
                        Need help with finding the name of your Village/Neighborhood? You can
                        <a href="https://www.sec.state.ma.us/divisions/cis/historical/archaic-names.htm"
                           target="_blank"
                           rel="noopener">
                          view them here.
                        </a>
                      </p>
                    </ng-template>
                  </kendo-popover>
                </label>
                <kendo-dropdownlist [data]="stateOptions" [valuePrimitive]="true" formControlName="state">
                </kendo-dropdownlist>
                <div class="error-message">
                  <span *ngIf="submittedAddress && addressForm.get('residenceAddress.state')?.hasError('required')">
                    State is required.
                  </span>
                </div>
              </kendo-formfield>
              <ng-container *ngIf="cityOptions.length && addressForm.get('residenceAddress.state')?.value === 'Massachusetts'; else resCityInput">
                <kendo-formfield class="flex-item">
                  <label kendoLabel class="label-and-asterisk">
                    City / Town <sup class="text-danger" [class.disable-super]="preferred==='Business'">*</sup>
                  </label>
                  <kendo-dropdownlist formControlName="cityTown"
                                      [data]="cityOptions"
                                      [valuePrimitive]="true"
                                      textField="value"
                                      valueField="value"
                                      class="dynamic-dropdown">
                  </kendo-dropdownlist>
                  <div class="error-message">
                    <span *ngIf="submittedAddress && addressForm.get('residenceAddress.cityTown')?.hasError('required')">
                      City/Town is required.
                    </span>
                  </div>
                </kendo-formfield>
              </ng-container>

              <ng-template #resCityInput>
                <kendo-formfield class="flex-item">
                  <label kendoLabel class="label-and-asterisk">
                    City / Town <sup class="text-danger" [class.disable-super]="preferred==='Business'">*</sup>
                  </label>
                  <input kendoTextBox formControlName="cityTown" class="city-town-input-element" />
                  <div class="error-message">
                    <span *ngIf="submittedAddress && addressForm.get('residenceAddress.cityTown')?.hasError('required')">
                      City/Town is required.
                    </span>
                  </div>
                </kendo-formfield>
              </ng-template>
              <!-- wrap the overall label in a plain div -->
              <div class="flex-item zip-code-field">
                <label class="label-and-asterisk" kendoLabel>
                  Zip Code
                  <sup class="text-danger" [class.disable-super]="preferred==='Business'">*</sup>
                </label>

                <div class="zip-inputs">

                  <!-- form-field for the 5-digit part -->
                  <kendo-formfield>
                    <input kendoTextBox
                           formControlName="zipCode"
                           class="zip-input-boxes"
                           maxlength="5"
                           placeholder="12345"
                           (keypress)="allowOnlyNumbers($event)"
                           (paste)="onZipPaste($event,'residenceAddress.zipCode')" />
                    <!-- your existing error spans here -->
                    <div class="error-message">
                      <span *ngIf="submittedAddress && addressForm.get('residenceAddress.zipCode')?.hasError('required')">
                        Zip Code is required.
                      </span>
                      <span *ngIf="submittedAddress && addressForm.get('residenceAddress.zipCode')?.hasError('pattern')">
                        Must be exactly 5 digits.
                      </span>
                    </div>
                  </kendo-formfield>

                  <span class="hyphen">–</span>

                  <!-- form-field for the 4-digit plus-4 part -->
                  <kendo-formfield>
                    <input kendoTextBox
                           formControlName="zipPlus"
                           class="zip-input-boxes"
                           maxlength="4"
                           placeholder="6789"
                           (keypress)="allowOnlyNumbers($event)"
                           (paste)="onZipPaste($event,'residenceAddress.zipPlus')" />
                    <!-- error for just this control -->
                    <div class="error-message">
                      <span *ngIf="submittedAddress && addressForm.get('residenceAddress.zipPlus')?.hasError('pattern')">
                        Enter at least 2 digits (up to 4).
                      </span>
                    </div>
                  </kendo-formfield>

                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Row 3: Business Address -->
        <div class="form-row">
          <div class="flex-item" formGroupName="businessAddress">
            <div class="heading-and-error">
              <div class="section-heading">
                Business Address
              </div>
              <span class="error-message error-message-with-top-marg" *ngIf="submittedAddress && preferred === 'Residence' && addressForm.get('businessAddress')?.invalid">
                Please complete all the fields or none.
              </span>
            </div>

            <!-- Street -->
            <div class="label-and-error-message">
              <label class="label-and-asterisk" kendoLabel>
                Street Address
                <sup class="text-danger" [class.disable-super]="preferred==='Residence'">*</sup>
                <input type="checkbox"
                       kendoCheckBox
                       (change)="togglePoBox($event)"
                       [checked]="isPoBox"
                       class="checkbox-override"
                       [disabled]="!updateAddressChecked" />
                <label kendoLabel>
                  Is this a PO Box?
                </label>
              </label>
              <div class="error-message">
                <span *ngIf="submittedAddress && addressForm.get('businessAddress')?.hasError('noStreet')">
                  Street number and Address Line are required.
                </span>
              </div>
            </div>
            <div class="form-row-no-wrap"
                 *ngIf="!addressForm.get('businessAddress.isPoBox')!.value; else poBoxTpl">
              <input kendoTextBox formControlName="street1" id="street-number" placeholder="Street No." />
              <input kendoTextBox formControlName="street2" id="address-line" placeholder="Address Line" />
              <input kendoTextBox formControlName="street3" id="address-suffix" placeholder="Suffix" />
            </div>
            <ng-template #poBoxTpl>
              <div class="form-row-no-wrap">
                <!-- show PO Box literal -->
                <input kendoTextBox
                       formControlName="street2"
                       [readonly]="true"
                       [disabled]="true"
                       class="po-box-label"
                       placeholder="PO Box" />

                <!-- PO Box number -->
                <input kendoTextBox
                       formControlName="street3"
                       placeholder="PO Box number" />
              </div>
            </ng-template>
            <div class="error-message"
                 *ngIf="submittedAddress
             && addressForm.get('businessAddress.isPoBox')!.value
             && addressForm.get('businessAddress.street3')!.hasError('required')">
              Enter some PO Box number
            </div>
            <div class="form-row">
              <input kendoTextBox formControlName="street4" placeholder="Address Line 2" />
            </div>

            <!-- State / Zip -->
            <div class="form-row state-and-zip">
              <kendo-formfield class="flex-item">
                <label class="label-and-asterisk" kendoLabel>
                  State
                  <sup class="text-danger" [class.disable-super]="preferred==='Residence'">*</sup>
                  <button kendoPopoverAnchor
                          [popover]="pop"
                          showOn="hover"
                          class="village-hyperlink"
                          kendoButton>
                    ?
                  </button>

                  <kendo-popover #pop title="Need help?" [width]="400">
                    <!-- this template will override the `body` property -->
                    <ng-template kendoPopoverBodyTemplate>
                      <p>
                        Need help with finding the name of your Village/Neighborhood? You can
                        <a href="https://www.sec.state.ma.us/divisions/cis/historical/archaic-names.htm"
                           target="_blank"
                           rel="noopener">
                          view them here.
                        </a>
                      </p>
                    </ng-template>
                  </kendo-popover>
                </label>
                <kendo-dropdownlist [data]="stateOptions" [valuePrimitive]="true" formControlName="state">
                </kendo-dropdownlist>
                <div class="error-message">
                  <span *ngIf="submittedAddress && addressForm.get('businessAddress.state')?.hasError('required')">
                    State is required.
                  </span>
                </div>
              </kendo-formfield>
              <ng-container *ngIf="cityOptions.length && addressForm.get('businessAddress.state')?.value === 'Massachusetts'; else busCityInput">
                <kendo-formfield class="flex-item">
                  <label kendoLabel class="label-and-asterisk">
                    City / Town <sup class="text-danger" [class.disable-super]="preferred==='Residence'">*</sup>
                  </label>
                  <kendo-dropdownlist formControlName="cityTown"
                                      [data]="cityOptions"
                                      [valuePrimitive]="true"
                                      textField="value"
                                      valueField="value"
                                      class="dynamic-dropdown">
                  </kendo-dropdownlist>
                  <div class="error-message">
                    <span *ngIf="submittedAddress && addressForm.get('businessAddress.cityTown')?.hasError('required')">
                      City/Town is required.
                    </span>
                  </div>
                </kendo-formfield>
              </ng-container>

              <ng-template #busCityInput>
                <kendo-formfield class="flex-item">
                  <label kendoLabel class="label-and-asterisk">
                    City / Town <sup class="text-danger" [class.disable-super]="preferred==='Residence'">*</sup>
                  </label>
                  <input kendoTextBox formControlName="cityTown" class="city-town-input-element" />
                  <div class="error-message">
                    <span *ngIf="submittedAddress && addressForm.get('businessAddress.cityTown')?.hasError('required')">
                      City/Town is required.
                    </span>
                  </div>
                </kendo-formfield>
              </ng-template>
              <!-- wrap the overall label in a plain div -->
              <div class="flex-item zip-code-field">
                <label class="label-and-asterisk" kendoLabel>
                  Zip Code
                  <sup class="text-danger" [class.disable-super]="preferred==='Residence'">*</sup>
                </label>

                <div class="zip-inputs">

                  <!-- form-field for the 5-digit part -->
                  <kendo-formfield>
                    <input kendoTextBox
                           formControlName="zipCode"
                           class="zip-input-boxes"
                           maxlength="5"
                           placeholder="12345"
                           (keypress)="allowOnlyNumbers($event)"
                           (paste)="onZipPaste($event,'businessAddress.zipCode')" />
                    <!-- your existing error spans here -->
                    <div class="error-message">
                      <span *ngIf="submittedAddress && addressForm.get('businessAddress.zipCode')?.hasError('required')">
                        Zip Code is required.
                      </span>
                      <span *ngIf="submittedAddress && addressForm.get('businessAddress.zipCode')?.hasError('pattern')">
                        Must be exactly 5 digits.
                      </span>
                    </div>
                  </kendo-formfield>

                  <span class="hyphen">–</span>

                  <!-- form-field for the 4-digit plus-4 part -->
                  <kendo-formfield>
                    <input kendoTextBox
                           formControlName="zipPlus"
                           class="zip-input-boxes"
                           maxlength="4"
                           placeholder="6789"
                           (keypress)="allowOnlyNumbers($event)"
                           (paste)="onZipPaste($event,'businessAddress.zipPlus')" />
                    <!-- error for just this control -->
                    <div class="error-message">
                      <span *ngIf="submittedAddress && addressForm.get('businessAddress.zipPlus')?.hasError('pattern')">
                        Enter at least 2 digits (up to 4).
                      </span>
                    </div>
                  </kendo-formfield>

                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </form>
  <!-- Single Save Button -->
  <div class="buttons-row">
    <button kendoButton (click)="onCancel()" class="custom-button-alt">Cancel</button>
    <button kendoButton (click)="onSubmit()" [disabled]="!updatePersonalChecked && !updateAddressChecked" class="search-button">Submit</button>
  </div>
</div>
ts file: 
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import mockData from '../../../../mockdata/mock2.json';
import mockProfileData from '../../../../mockdata/mock-profile-data.json';
import { AbstractControl, FormBuilder, FormGroup, ValidationErrors, ValidatorFn, Validators } from '@angular/forms';
import { RefGetterService } from '../../../services/helper-services/ref-get-service/ref-getter.service';
import { CityCountyRef, SalutationRef, StateRef } from '../../../models/ref-items-model/ref-items.model';
import { take } from 'rxjs';
import { BuildEditObject, UpdatePersonalProfileInformation } from '../../../models/update-profile-info-model/update-profile-info.model';
import { UpdateNotaryProfileInformationService } from '../../../services/update-notary-profile-info/update-notary-profile-information.service';

@Component({
    selector: 'app-update-notary-profile-info',
    templateUrl: './update-notary-profile-info.component.html',
    styleUrls: ['./update-notary-profile-info.component.css', '../add-new-record/add-new-record.component.css']
})
export class UpdateNotaryProfileInfoComponent implements OnInit {
    public personalForm!: FormGroup;
    public addressForm!: FormGroup;
    showValidationErrors: boolean = false;
    // start empty, we’ll overwrite once the call returns
    public salutationOptions: string[] = [];
    public salutationRefs: SalutationRef[] = [];
    public stateOptions: string[] = [];
    public stateRefs: StateRef[] = [];
    public cityOptions: CityCountyRef[] = [];
    private allCityCountyData: CityCountyRef[] = [];

    public updatePersonalChecked = false;
    public updateAddressChecked = false;
    public isPoBox = false;
    public submittedAddress = false;

    private originalPersonalValues: any;
    private originalAddressValues: any;

    public displayData?: BuildEditObject;

    constructor(
        private fb: FormBuilder,
        private refGetter: RefGetterService,
        private router: Router,
        private route: ActivatedRoute,
        private updateService: UpdateNotaryProfileInformationService
    ) {
        // 1) Preferred: getCurrentNavigation (only on the initial nav)
        const nav = this.router.getCurrentNavigation();
        this.displayData = nav?.extras.state?.['buildEditObject'] as BuildEditObject | undefined;

        // 2) Fallback (and works after page reload):
        if (!this.displayData && history.state?.buildEditObject) {
            this.displayData = history.state.buildEditObject;
        }

        console.log('Received buildEditObject:', this.displayData);
    }

    ngOnInit() {
        // — your existing route + mock-data lookup logic —
        const idParam = this.route.snapshot.paramMap.get('id');
        if (!idParam) { console.error('No "id" in route!'); return; }
        const notaryId = Number(idParam);
        if (isNaN(notaryId)) { console.error(`Bad id: ${idParam}`); return; }

        // build your form with all controls starting as null
        this.personalForm = this.fb.group({
            salutation: [null],
            firstName: [null],
            middleName: [null],
            lastName: [null],
            suffix: [null],
            dateOfBirth: [null],
            dateOfDeath: [null],
            dateOfResignation: [null], 
            email: [null],
            primaryPhone1: [null],
            primaryPhone2: [null],
            primaryPhone3: [null],
            secondaryPhone1: [null],
            secondaryPhone2: [null],
            secondaryPhone3: [null],
            changeType: [null,[Validators.required]]
        });

        this.buildAddressForm();


        if (this.displayData?.personalInfoDetails) {
            const p = this.displayData.personalInfoDetails;

            // 1) Salutation, names, suffix
            this.personalForm.patchValue({
                salutation: p.salutationType,    // string-based dropdown
                firstName: p.firstName,
                middleName: p.middleName,
                lastName: p.lastName,
                suffix: p.suffix,

                // 2) Dates
                dateOfBirth: p.dateOfBirth ? new Date(p.dateOfBirth) : null,
                dateOfDeath: p.dateOfDeath ? new Date(p.dateOfDeath) : null,
                dateOfResignation: p.dateOfResignation ? new Date(p.dateOfResignation) : null,

                // 3) Email
                email: p.emailAddress || null
            });

            // 4) Phones: strip non-digits, then slice into [3,3,4]
            const toSlices = (raw: string) => {
                const d = raw.replace(/\D/g, '');
                return [d.slice(0, 3), d.slice(3, 6), d.slice(6)];
            };

            const primary = p.contactDetails.find(c =>
                c.contactType === 'Phone' && c.isPrimary
            );
            if (primary) {
                const [a, b, c] = toSlices(primary.contactValue);
                this.personalForm.patchValue({
                    primaryPhone1: a,
                    primaryPhone2: b,
                    primaryPhone3: c
                });
            }

            const secondary = p.contactDetails.find(c =>
                c.contactType === 'Phone' && !c.isPrimary
            );
            if (secondary) {
                const [a, b, c] = toSlices(secondary.contactValue);
                this.personalForm.patchValue({
                    secondaryPhone1: a,
                    secondaryPhone2: b,
                    secondaryPhone3: c
                });
            }

            // 5) Type-of-change left null → radios start unchecked
            //    (and dateOfResignation already set to null above)
        }

        this.originalPersonalValues = this.personalForm.value;
        this.personalForm.disable();
        //this.originalAddressValues = this.addressForm.value;
        //this.addressForm.disable();

        const resState = this.addressForm.get('residenceAddress.state')!;
        const busState = this.addressForm.get('businessAddress.state')!;
        resState.valueChanges.subscribe(() => {
            this.addressForm.get('residenceAddress.cityTown')!.reset();
            this.updateCityOptions();
        });
        busState.valueChanges.subscribe(() => {
            this.addressForm.get('businessAddress.cityTown')!.reset();
            this.updateCityOptions();
        });

        // now fetch the salutations, map to the "value" array, or log any error
        //this.refGetter.getSalutations().subscribe(
        //  (all: SalutationRef[]) => {
        //    this.salutationOptions = all.map(x => x.value);
        //  },
        //  err => console.error('Failed to load salutations', err)
        //);

        this.refGetter.getSalutations().subscribe(
            (all: SalutationRef[]) => {
                this.salutationRefs = all;                    //  save full array
                this.salutationOptions = all.map(x => x.value);
            },
            err => console.error('Failed to load salutations', err)
        );

        this.getStatesForDropDown();
        this.cacheCityCountyData();
    }

    private buildAddressForm() {
        this.addressForm = this.fb.group({
            preferredAddress: ['Residence', Validators.required],

            residenceAddress: this.fb.group({
                street1: [''],
                street2: [''],
                street3: [''],
                street4: [''],
                state: [''],
                zipCode: ['', [Validators.required, Validators.pattern(/^\d{5}$/)]],
                zipPlus: ['', Validators.pattern(/^$|^\d{2,4}$/)],
                cityTown: ['']
            }, { validators: this.streetValidator() }),

            businessAddress: this.fb.group({
                isPoBox: [false],          // new!
                street1: [''],             // Street No.
                street2: [''],             // Address Line / will become "PO Box"
                street3: [''],             // Suffix / will become PO Box number
                street4: [''],             // Address Line 2
                state: [''],
                zipCode: ['', [Validators.required, Validators.pattern(/^\d{5}$/)]],
                zipPlus: ['', Validators.pattern(/^$|^\d{2,4}$/)],
                cityTown: ['']
            }, { validators: this.streetValidator() })
        });

        // re-apply validators when preferred changes
        this.addressForm.get('preferredAddress')!
            .valueChanges
            .subscribe(pref => this.updateAddressValidators(pref));

        // init validators for default “Residence”
        this.updateAddressValidators(this.addressForm.get('preferredAddress')!.value);
    }

    private streetValidator(): ValidatorFn {
        return (group: AbstractControl): ValidationErrors | null => {
            const g = group as FormGroup;
            const s1 = g.get('street1')!.value;
            const s2 = g.get('street2')!.value;
            return (!s1 && !s2) ? { noStreet: true } : null;
        };
    }

    private optionalGroupValidator(): ValidatorFn {
        return (group: AbstractControl): ValidationErrors | null => {
            const g = group as FormGroup;
            const hasAny = Object.values(g.value).some(v => !!v);
            if (!hasAny) { return null; }

            // must satisfy street + required fields
            const streetErr = this.streetValidator()(g);
            if (streetErr) { return { noStreet: true }; }
            for (const field of ['state', 'zipCode', 'cityTown']) {
                if (!g.get(field)!.value) {
                    return { [`${field}Req`]: true };
                }
            }
            return null;
        };
    }

    private updateAddressValidators(pref: 'Residence' | 'Business') {
        const resG = this.addressForm.get('residenceAddress') as FormGroup;
        const busG = this.addressForm.get('businessAddress') as FormGroup;

        // clear all
        [resG, busG].forEach(grp => {
            grp.clearValidators();
            Object.values(grp.controls).forEach(c => c.clearValidators());
        });

        if (pref === 'Residence') {
            // Residence required
            resG.setValidators(this.streetValidator());
            ['state', 'zipCode', 'cityTown'].forEach(name => {
                const ctrl = resG.get(name)!;
                ctrl.setValidators(name === 'zipCode'
                    ? [Validators.required, Validators.pattern(/^\d{5}$/)]
                    : Validators.required);
            });
            resG.get('zipPlus')!.setValidators(Validators.pattern(/^$|^\d{2,4}$/));

            // Business optional
            busG.setValidators(this.optionalGroupValidator());
        } else {
            // Business required
            busG.setValidators(this.streetValidator());
            ['state', 'zipCode', 'cityTown'].forEach(name => {
                const ctrl = busG.get(name)!;
                ctrl.setValidators(name === 'zipCode'
                    ? [Validators.required, Validators.pattern(/^\d{5}$/)]
                    : Validators.required);
            });
            busG.get('zipPlus')!.setValidators(Validators.pattern(/^$|^\d{2,4}$/));

            // Residence optional
            resG.setValidators(this.optionalGroupValidator());
        }

        // re-run
        [resG, busG].forEach(g => g.updateValueAndValidity({ onlySelf: true }));
        ['state', 'zipCode', 'cityTown', 'zipPlus']
            .forEach(n => {
                resG.get(n)?.updateValueAndValidity({ onlySelf: true });
                busG.get(n)?.updateValueAndValidity({ onlySelf: true });
            });
    }

    public togglePoBox(event: any): void {
        const isPo = event.target.checked;
        const bus = this.addressForm.get('businessAddress') as FormGroup;
        bus.get('isPoBox')!.setValue(isPo);

        if (isPo) {
            // 1) Clear street1, street4
            bus.get('street1')!.setValue(null);
            bus.get('street1')!.clearValidators();
            bus.get('street1')!.updateValueAndValidity();
            bus.get('street4')!.setValue(null);
            bus.get('street4')!.clearValidators();
            bus.get('street4')!.updateValueAndValidity();

            // 2) Force street2 = "PO Box" (keep enabled but read-only in template)
            bus.get('street2')!.setValue('PO Box');
            bus.get('street2')!.clearValidators();
            bus.get('street2')!.updateValueAndValidity();

            // 3) Make PO Box number (street3) required
            bus.get('street3')!
                .setValidators([Validators.required]);
            bus.get('street3')!.updateValueAndValidity();

            // 4) Replace the group-level validator so only street3 is enforced
            bus.clearValidators();
            bus.setValidators(this.poBoxGroupValidator());
            bus.updateValueAndValidity();
        } else {
            // Reset to the default “Business required” rules
            bus.get('street2')!.setValue('');
            bus.get('street3')!.setValue('');
            bus.get('street1')!.setValidators([]);
            bus.get('street2')!.setValidators([]);
            bus.get('street3')!.setValidators([]);
            bus.get('street4')!.setValidators([]);
            bus.get('street1')!.updateValueAndValidity();
            bus.get('street2')!.updateValueAndValidity();
            bus.get('street3')!.updateValueAndValidity();
            bus.get('street4')!.updateValueAndValidity();

            // re-apply the original Business validators
            this.updateAddressValidators('Business');
        }
    }

    private poBoxGroupValidator(): ValidatorFn {
        return (ctl: AbstractControl): ValidationErrors | null => {
            const g = ctl as FormGroup;
            return g.get('street3')!.value
                ? null
                : { poBoxNumberReq: true };
        };
    }

    private getStatesForDropDown() {
        this.refGetter.getStates().pipe(take(1)).subscribe(
            (all: StateRef[]) => {
                this.stateRefs = all;               // store full refs
                this.stateOptions = all.map(s => s.value);

                // now that we know full state list, patch the addressForm:
                this.patchAddressDetails();
            },
            err => console.error('States load failed', err)
        );
    }

    private cacheCityCountyData() {
        this.refGetter.getCityCounty().pipe(take(1)).subscribe(
            (all: CityCountyRef[]) => {
                this.allCityCountyData = all;
                this.updateCityOptions();

                // now that cityOptions is populated, re-patch the cityTown field
                if (this.displayData) {
                    const addrs = this.displayData.personalInfoDetails.addressDetails;
                    const res = addrs.find(a => a.addressType === 'Residential');
                    if (res) {
                        this.addressForm.get('residenceAddress.cityTown')!
                            .setValue(res.city);
                    }
                    const bus = addrs.find(a => a.addressType === 'Business');
                    if (bus) {
                        this.addressForm.get('businessAddress.cityTown')!
                            .setValue(bus.city);
                    }
                }
            },
            err => console.error('CityCounty load failed', err)
        );
    }

    private patchAddressDetails(): void {
        if (!this.displayData) { return; }
        const addrs = this.displayData.personalInfoDetails.addressDetails;

        // 1) Pick out the preferred vs. non-preferred
        const preferred = addrs.find(a => a.isPrefered) || addrs[0];
        const other = addrs.find(a => !a.isPrefered) || null;

        // 2) Select the radio
        const prefType = preferred.addressType === 'Business' ? 'Business' : 'Residence';
        this.addressForm.get('preferredAddress')!.setValue(prefType);

        // 3) Helper to patch a single FormGroup
        const doPatch = (grpName: 'residenceAddress' | 'businessAddress', addr: any) => {
            const grp = this.addressForm.get(grpName) as FormGroup;

            // State → full name lookup
            const stateRef = this.stateRefs.find(s => s.stateId === addr.stateId);
            const fullState = stateRef?.value || '';
            grp.get('state')!.setValue(fullState);

            // trigger city-options (will pick up MA → dropdown)
            grp.get('cityTown')!.reset();
            this.updateCityOptions();
            grp.get('cityTown')!.setValue(addr.city);

            // Zip
            grp.get('zipCode')!.setValue(addr.zipCode);
            grp.get('zipPlus')!.setValue(addr.zipPlus || '');

            if (grpName === 'businessAddress' && addr.isPoBox) {
                // Set the checkbox state first
                this.isPoBox = true;
                // let your helper do the heavy lifting
                this.togglePoBox({ target: { checked: true } });
                // PO Box number → street3
                grp.get('street3')!.setValue(addr.aptNumber);
            } else {
                // ensure PO-Box logic turned off for business
                if (grpName === 'businessAddress') {
                    this.isPoBox = false;
                    this.togglePoBox({ target: { checked: false } });
                }
                grp.patchValue({
                    street1: addr.streetNumber,
                    street2: addr.streetName,
                    street3: addr.aptNumber,
                    street4: addr.addressLine2
                });
            }
        };

        // 4) Patch “Residence” if present
        const res = addrs.find(a => a.addressType === 'Residential');
        if (res) { doPatch('residenceAddress', res); }

        // 5) Patch “Business” if present
        const bus = addrs.find(a => a.addressType === 'Business');
        if (bus) { doPatch('businessAddress', bus); }

        // 6) Snapshot these new values as “original” and re-disable
        this.originalAddressValues = this.addressForm.value;
        this.addressForm.disable();
    }

    private updateCityOptions() {
        const res = this.addressForm.get('residenceAddress.state')!.value;
        const bus = this.addressForm.get('businessAddress.state')!.value;
        this.cityOptions = (res === 'Massachusetts' || bus === 'Massachusetts')
            ? this.allCityCountyData
            : [];
    }

    public get preferred(): 'Residence' | 'Business' {
        return this.addressForm.get('preferredAddress')!.value;
    }

    public onCheckboxChange(value: boolean): void {
        console.log('Checkbox value:', value);
    }

    public togglePersonalInfo(evt: Event): void {
        const chk = (evt.target as HTMLInputElement).checked;
        this.updatePersonalChecked = chk;

        if (chk) {
            // enable editing
            this.personalForm.enable();
        } else {
            // reset to original values, then disable
            this.personalForm.reset(this.originalPersonalValues);
            this.personalForm.disable();
        }
    }

    public toggleAddressInfo(evt: Event): void {
        const chk = (evt.target as HTMLInputElement).checked;
        this.updateAddressChecked = chk;

        //console.log('Toggle checkbox:', chk);
        //console.log('Before toggle - Residence city:', this.addressForm.get('residenceAddress.cityTown')?.value);
        //console.log('Before toggle - Business city:', this.addressForm.get('businessAddress.cityTown')?.value);

        if (chk) {
            // Enable the form first
            this.addressForm.enable();

            //console.log('After enable - Residence city:', this.addressForm.get('residenceAddress.cityTown')?.value);
            //console.log('After enable - Business city:', this.addressForm.get('businessAddress.cityTown')?.value);

            // Then restore city values for Massachusetts states
            this.restoreCityValuesForMassachusetts();
        } else {
            // Reset to original values, then disable
            this.addressForm.reset(this.originalAddressValues);

            //console.log('After reset - Residence city:', this.addressForm.get('residenceAddress.cityTown')?.value);
            //console.log('After reset - Business city:', this.addressForm.get('businessAddress.cityTown')?.value);

            // IMPORTANT: After reset, we need to update city options and restore city values
            // because the state-to-city dependency needs to be reestablished
            this.updateCityOptions();

            // Re-patch the city values after cityOptions is updated
            this.restoreCityValuesForMassachusetts();

            this.addressForm.disable();
        }
    }

    private restoreCityValuesForMassachusetts(): void {
        if (!this.displayData) return;

        const addrs = this.displayData.personalInfoDetails.addressDetails;

        // Use a longer timeout to ensure dropdowns are fully rendered
        setTimeout(() => {
            // Restore residence city if it exists and state is Massachusetts
            const res = addrs.find(a => a.addressType === 'Residential');
            if (res && this.addressForm.get('residenceAddress.state')?.value === 'Massachusetts') {
                const cityControl = this.addressForm.get('residenceAddress.cityTown');
                //console.log('Setting residence city to:', res.city);
                //console.log('City options available:', this.cityOptions);
                if (cityControl) {
                    cityControl.setValue(res.city, { emitEvent: false });
                    //console.log('After setting residence city:', cityControl.value);
                }
            }

            // Restore business city if it exists and state is Massachusetts  
            const bus = addrs.find(a => a.addressType === 'Business');
            if (bus && this.addressForm.get('businessAddress.state')?.value === 'Massachusetts') {
                const cityControl = this.addressForm.get('businessAddress.cityTown');
                //console.log('Setting business city to:', bus.city);
                if (cityControl) {
                    cityControl.setValue(bus.city, { emitEvent: false });
                    //console.log('After setting business city:', cityControl.value);
                }
            }
        }, 100); // Increased timeout
    }

    public allowOnlyNumbers(event: KeyboardEvent) {
        if (event.key.length === 1 && !/^\d$/.test(event.key)) event.preventDefault();
    }

    public onZipPaste(event: ClipboardEvent, controlPath: string) {
        event.preventDefault();
        const digits = (event.clipboardData?.getData('text') || '').replace(/\D/g, '');
        this.addressForm.get(controlPath)?.setValue(digits);
    }

    markAllFieldsTouched(formGroup: FormGroup): void {
        Object.keys(formGroup.controls).forEach((key) => {
            const control = formGroup.get(key);
            control?.markAsTouched();
            control?.updateValueAndValidity();
        });
    }

    public onSubmit(): void {
        if (!this.updatePersonalChecked && !this.updateAddressChecked) {
            return;
        }

        this.showValidationErrors = true;


        if (this.updatePersonalChecked) {
            if (this.personalForm.invalid) {
            this.markAllFieldsTouched(this.personalForm); // highlight errors
            return; // stop submission
        }
            if (this.personalForm.valid) {
                const payload = this.buildUpdatePersonalPayload();
                console.log('UpdatePersonalProfileInformation:', payload);
                this.updateService.updatePersonalInformation(payload)
                    .subscribe({
                        next: success => {
                            console.log('UpdatePersonalProfileInformation success:', success);
                            this.router.navigate(['/notary-profile', payload.applicantId]);
                            // maybe show a toast or navigate on success
                        },
                        error: err => {
                            console.error('UpdatePersonalProfileInformation failed:', err);
                            // handle/display errors
                        }
                    });
            } else {
                console.warn('Personal form is invalid:', this.personalForm.errors);
            }
        }
        if (this.updateAddressChecked) {
            this.submittedAddress = true;
            if (this.addressForm.valid) {
                console.log('Address Information:', this.addressForm.value);
            } else {
                console.warn('Address form is invalid:', this.addressForm.errors);
            }
        }
    }

    private buildUpdatePersonalPayload(): UpdatePersonalProfileInformation {
        const f = this.personalForm.value;
        const d = this.displayData!.personalInfoDetails;
        const applicantId = d.applicantId;
        const accountId = d.accountId;

        // 1) lookup salutationTypeId
        const sel = f.salutation as string;
        const salRef = this.salutationRefs.find(r => r.value === sel);
        const salutationTypeId = salRef?.salutationTypeId ?? 0;

        // 2) normalize name fields
        const norm = (v: string | null) => v ? v : null;

        // 3) format dates
        const formatDate = (dt: Date | null): string | null => {
            if (!dt) {
                return null;
            }
            const year = dt.getFullYear();
            const month = String(dt.getMonth() + 1).padStart(2, '0');
            const day = String(dt.getDate()).padStart(2, '0');
            return `${year}-${month}-${day}T00:00:00`;
        };
        const dob = formatDate(f.dateOfBirth);
        const dod = formatDate(f.dateOfDeath);
        const res = formatDate(f.dateOfResignation);

        // 4) change flags
        const isOfficial = f.changeType === 'Official';
        const isCorrection = f.changeType === 'Correction';

        // 5) build contacts array
        const contacts: UpdatePersonalProfileInformation['contactsDto'] = [];
        const makePhone = (blocks: [string, string, string], primary: boolean) => {
            const val = blocks.join('');
            if (!val) return;  // skip if totally blank
            const orig = d.contactDetails.find(c =>
                c.contactType === 'Phone' && c.isPrimary === primary
            );
            contacts.push({
                contactId: orig?.contactId ?? 0,
                applicantId,
                contactTypeId: 1,
                contactValue: val,
                isPrimary: primary
            });
        };
        makePhone([f.primaryPhone1, f.primaryPhone2, f.primaryPhone3], true);
        makePhone([f.secondaryPhone1, f.secondaryPhone2, f.secondaryPhone3], false);

        // 6) email
        const emailVal = (f.email as string || '').trim();
        if (emailVal) {
            const orig = d.contactDetails.find(c => c.contactType === 'Email');
            contacts.push({
                contactId: orig?.contactId ?? 0,
                applicantId,
                contactTypeId: 2,
                contactValue: emailVal,
                isPrimary: false
            });
        }

        // 7) put it all together
        return {
            applicantId,
            accountId,
            salutationTypeId,
            firstName: f.firstName,
            middleName: norm(f.middleName),
            lastName: f.lastName,
            suffix: norm(f.suffix),
            dateOfBirth: dob!,
            dateOfDeath: dod!,
            resignationDate: res!,
            isOfficalNameChange: isOfficial,
            isNameCorrection: isCorrection,
            contactsDto: contacts
        };
    }

    public onCancel(): void {
        // pull the applicantId out of your displayData
        const applicantId = this.displayData?.personalInfoDetails?.applicantId;

        // navigate back to /notary-profile/:id
        if (applicantId != null) {
            this.router.navigate(['/notary-profile', applicantId]);
        } else {
            console.error('No applicantId available to navigate back.');
        }
    }
}
update-notary-profile-information.service.ts file:
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { environment } from '../../../environments/environment';
import { Observable } from 'rxjs';
import { UpdatePersonalProfileInformation } from '../../models/update-profile-info-model/update-profile-info.model';

@Injectable({
  providedIn: 'root'
})
export class UpdateNotaryProfileInformationService {
  private apiUrl = `${environment.apiurl}/api/InternalUser/UpdatePersonalInformation`;

  constructor(private http: HttpClient) { }

  /**
   * Sends the DTO to the backend.
   * Returns an Observable that emits `true` when the update succeeds.
   */
  updatePersonalInformation(
    payload: UpdatePersonalProfileInformation
  ): Observable<boolean> {
    return this.http.post<boolean>(this.apiUrl, payload);
  }
}
Interface: 
export interface BuildEditObject {
  personalInfoDetails: {
    accountId: number | null,
    emailAddress: string,
    salutationTypeId: number,
    salutationType: string,
    firstName: string,
    middleName: string,
    lastName: string,
    suffix: string,
    dateOfBirth: string,
    applicantId: number,
    notaryIdentifier: number,
    dateOfDeath: string,
    dateOfResignation: string | null;
    addressDetails: [
      {
        addressId: number,
        applicantId: number,
        addressTypeId: number,
        addressType: string,
        isPrefered: boolean,
        isPoBox: boolean,
        streetNumber: string,
        streetName: string,
        aptNumber: string,
        addressLine2: string,
        zipCode: string,
        zipPlus: string,
        city: string,
        county: string,
        district: string,
        stateId: number,
        state: string,
      }
    ],
    contactDetails: [
      {
        contactTypeId: number,
        contactType: string,
        contactId: number,
        applicantId: number,
        contactValue: string,
        isPrimary: boolean
      }
    ]
  }
}
