Alright. Tricky requirement. Carefully go through the image and the files. 
As shown in the image.. the input elements have a certain type of placeholder color (grey ish) but the salutation and dob fields have the 
placeholder color as black? Fix that? 
add-new-record.component.html: 
<div class="page-wrapper">
  <div class="first-row"><h2>{{viewHeading}}</h2><div class="required-indicator"><div class="asterisk">*</div><div class="required-indicator-text"> - Required fields</div></div></div>
  <!--<button kendoButton (click)="testDupeMatchDialog()">Invoke Dupe Match Dialog</button>-->
  <form [formGroup]="form" (ngSubmit)="onSubmit()" *ngIf="!showAddressSection">
    <!-- Row 1: Salutation / First / Middle / Last / Suffix -->
    <div class="form-row">
      <kendo-formfield class="flex-item">
        <label kendoLabel for="salutation">
          Salutation <sup class="text-danger">*</sup>
        </label>
        <kendo-dropdownlist [data]="salutationOptions"
                            formControlName="salutation"
                            id="salutation"
                            [valuePrimitive]="true"
                            [defaultItem]="{ text: 'Select...', value: '' }"
                            textField="text"
                            valueField="value">
        </kendo-dropdownlist>
        <div class="error-message">
          <small *ngIf="submitted && form.get('salutation')?.hasError('required')">
            Salutation is required.
          </small>
        </div>
      </kendo-formfield>

      <kendo-formfield class="flex-item">
        <label kendoLabel for="firstName">
          First Name <sup class="text-danger">*</sup>
        </label>
        <input kendoTextBox id="firstName" formControlName="firstName" placeholder="Enter First Name" />
        <div class="error-message">
          <small *ngIf="submitted && form.get('firstName')?.hasError('required')">
            First Name is required.
          </small>
          <small *ngIf="submitted && form.get('firstName')?.hasError('pattern')">
            Only letters, hyphens or “/” allowed.
          </small>
        </div>
      </kendo-formfield>

      <kendo-formfield class="flex-item">
        <label kendoLabel for="middleName">Middle Name / Initial <sup class="text-danger disable-super">*</sup>
        </label>
        <input kendoTextBox id="middleName" formControlName="middleName" placeholder="Enter Middle Name" />
        <div class="error-message">
          <small *ngIf="submitted && form.get('middleName')?.hasError('pattern')">
            Only letters, hyphens or “/” allowed.
          </small>
        </div>
      </kendo-formfield>

      <kendo-formfield class="flex-item">
        <label kendoLabel for="lastName">
          Last Name <sup class="text-danger">*</sup>
        </label>
        <input kendoTextBox id="lastName" formControlName="lastName" placeholder="Enter Last Name" />
        <div class="error-message">
          <small *ngIf="submitted && form.get('lastName')?.hasError('required')">
            Last Name is required.
          </small>
          <small *ngIf="submitted && form.get('lastName')?.hasError('pattern')">
            Only letters, hyphens or “/” allowed.
          </small>
        </div>
      </kendo-formfield>

      <kendo-formfield class="flex-item">
        <label kendoLabel for="suffix">Suffix<sup class="text-danger disable-super">*</sup>
        </label>
        <input kendoTextBox id="suffix" placeholder="Jr, II" formControlName="suffix" />
        <div class="error-message">
          <small *ngIf="submitted && form.get('suffix')?.hasError('pattern')">
            Only letters, hyphens or “/” allowed.
          </small>
        </div>
      </kendo-formfield>
    </div>

    <!-- Row 2: Date of Birth/ Email -->
    <div class="form-row">
      <kendo-formfield class="flex-item">
        <label kendoLabel for="dateOfBirth">
          Date of Birth<sup class="text-danger">*</sup>
        </label>
        <kendo-datepicker formControlName="dateOfBirth"
                          id="dateOfBirth"
                          format="MM/dd/yyyy"
                          [formatPlaceholder]="{ month: 'MM', day: 'DD', year: 'YYYY' }"
                          [min]="minDob"
                          [max]="maxDob">
        </kendo-datepicker>
        <div class="error-message">
            <small *ngIf="submitted && form.get('dateOfBirth')?.hasError('required')">
                Date of Birth is required.
            </small>
            <small *ngIf="submitted && form.get('dateOfBirth')?.errors?.['minAge']">
                You must be at least {{ form.get('dateOfBirth')?.errors?.['minAge'].required }} years old.
                (Your age: {{ form.get('dateOfBirth')?.errors?.['minAge'].actual }})
            </small>
        </div>
      </kendo-formfield>
      <kendo-formfield class="flex-item">
        <label kendoLabel for="email">
          Email Address <sup class="text-danger">*</sup>
        </label>
        <input kendoTextBox id="email" formControlName="email" placeholder="example@abc.com" />
        <div class="error-message">
          <small *ngIf="submitted && form.get('email')?.hasError('required')">
            Email is required.
          </small>
          <small *ngIf="submitted && form.get('email')?.hasError('email')">
            Invalid email format.
          </small>
        </div>
      </kendo-formfield>
    </div>

    <!-- Row 3:Phones -->
    <div class="form-row">
      <!-- Primary -->
      <div class="flex-item">
        <label class="group-label">
          Primary Phone <sup class="text-danger">*</sup>
        </label>

        <div class="phone-group">
          <kendo-formfield>
            <input kendoTextBox
                   formControlName="primaryPhone1"
                   maxlength="3"
                   class="three-size"
                   placeholder="---"
                   #pp1
                   (input)="onPhoneInput(pp1, pp2)"
                   (keydown)="onPhoneKeydown($event, pp1, null)"
                   (paste)="onPhonePaste($event, [pp1, pp2, pp3])"
                   />
          </kendo-formfield>
          <span class="dash">-</span>
          <kendo-formfield>
            <input kendoTextBox
                   formControlName="primaryPhone2"
                   maxlength="3"
                   class="three-size"
                   placeholder="---"
                   #pp2
                   (input)="onPhoneInput(pp2, pp3)"
                   (keydown)="onPhoneKeydown($event, pp2, pp1)"
                   (paste)="onPhonePaste($event, [pp1, pp2, pp3])"
                   />
          </kendo-formfield>
          <span class="dash">-</span>
          <kendo-formfield>
            <input kendoTextBox
                   formControlName="primaryPhone3"
                   maxlength="4"
                   class="four-size"
                   placeholder="----"
                   #pp3
                   (input)="onPhoneInput(pp3)"
                   (keydown)="onPhoneKeydown($event, pp3, pp2)"
                   (paste)="onPhonePaste($event, [pp1, pp2, pp3])"
                   />
          </kendo-formfield>
        </div>

        <div class="error-message">
          <small *ngIf="submitted && (
       form.get('primaryPhone1')?.invalid ||
       form.get('primaryPhone2')?.invalid ||
       form.get('primaryPhone3')?.invalid
    )">
            Invalid primary phone.
          </small>
        </div>
      </div>

      <!-- Secondary -->
      <div class="flex-item">
        <label class="group-label">
          Secondary Phone <sup class="text-danger disable-super">*</sup>
        </label>
        <div class="phone-group">
          <kendo-formfield>
            <input kendoTextBox
                   formControlName="secondaryPhone1"
                   maxlength="3"
                   class="three-size"
                   placeholder="---"
                   #pp4
                   (input)="onPhoneInput(pp4, pp5)"
                   (keydown)="onPhoneKeydown($event, pp4, null)"
                   (paste)="onPhonePaste($event, [pp4, pp5, pp6])"
                   />
          </kendo-formfield>
          <span class="dash">-</span>
          <kendo-formfield>
            <input kendoTextBox
                   formControlName="secondaryPhone2"
                   maxlength="3"
                   class="three-size"
                   placeholder="---"
                   #pp5
                   (input)="onPhoneInput(pp5, pp6)"
                   (keydown)="onPhoneKeydown($event, pp5, pp4)"
                   (paste)="onPhonePaste($event, [pp4, pp5, pp6])"
                   />
          </kendo-formfield>
          <span class="dash">-</span>
          <kendo-formfield>
            <input kendoTextBox
                   formControlName="secondaryPhone3"
                   maxlength="4"
                   class="four-size"
                   placeholder="----"
                   #pp6
                   (input)="onPhoneInput(pp6)"
                   (keydown)="onPhoneKeydown($event, pp6, pp5)"
                   (paste)="onPhonePaste($event, [pp4, pp5, pp6])"
                  />
          </kendo-formfield>
        </div>

        <div class="error-message">
          <small *ngIf="submitted && (
       form.get('secondaryPhone1')?.invalid ||
       form.get('secondaryPhone2')?.invalid ||
       form.get('secondaryPhone3')?.invalid
    )">
            Invalid secondary phone.
          </small>
        </div>
      </div>
    </div>

    <!-- Actions -->
    <div class="button-row">
      <button kendoButton type="button" class="btn-cancel" (click)="onCancel()">
        Cancel
      </button>
      <button kendoButton type="submit" class="btn-next">Next</button>
    </div>
  </form>

  <div *ngIf="showAddressSection" class="address-section">
    <form [formGroup]="addressForm" (ngSubmit)="onAddressSubmit()">
      <!-- Row 1: Preferred Radio -->
      <div class="form-row">
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
            <input kendoTextBox formControlName="street2" id="address-line"  placeholder="Street Name" />
            <input kendoTextBox formControlName="street3" id="address-suffix" placeholder="House No." />
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
                    <span *ngIf="submittedAddress && addressForm.get('residenceAddress.cityTown')?.hasError('pattern')">
                        Please enter a valid City/Town name.
                    </span>
                </div>
              </kendo-formfield>
            </ng-template>
            <!--<kendo-formfield class="flex-item zip-code-field">
    <label class="label-and-asterisk" kendoLabel>
      Zip Code <sup class="text-danger" [class.disable-super]="preferred==='Business'">*</sup>
    </label>
    <div class="zip-inputs">
      <input kendoTextBox
             formControlName="zipCode"
             class="zip-input-boxes"
             maxlength="5"
             placeholder="12345"
             (keypress)="allowOnlyNumbers($event)"
             (paste)="onZipPaste($event,'residenceAddress.zipCode')" />
      <span class="hyphen">–</span>
      <input kendoTextBox
             formControlName="zipPlus"
             class="zip-input-boxes"
             maxlength="4"
             placeholder="6789"
             (keypress)="allowOnlyNumbers($event)"
             (paste)="onZipPaste($event,'residenceAddress.zipPlus')" />
    </div>
    <div class="error-message">
      <span *ngIf="submittedAddress && addressForm.get('residenceAddress.zipCode')?.hasError('required')">
        Zip Code is required.
      </span>
      <span *ngIf="submittedAddress && addressForm.get('residenceAddress.zipCode')?.hasError('pattern')">
        Must be exactly 5 digits.
      </span>
      <span *ngIf="submittedAddress && addressForm.get('residenceAddress.zipPlus')?.hasError('pattern')">
        Enter at least 2 digits (up to 4).
      </span>
    </div>
  </kendo-formfield>-->
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
              <input type="checkbox" class="checkbox-override" kendoCheckBox formControlName="isPoBox" />
              <label kendoLabel>
                Is this a PO Box?
              </label>
            </label>
            <div class="error-message">
              <span *ngIf="submittedAddress && addressForm.get('businessAddress')?.hasError('noStreet') && !busPoBox">
                Street number and Address Line are required.
              </span>
            </div>
          </div>
          <div class="form-row-no-wrap" *ngIf="!busPoBox">
            <input kendoTextBox formControlName="street1" id="street-number" placeholder="Street No." />
            <input kendoTextBox formControlName="street2" id="address-line" placeholder="Address Line" />
            <input kendoTextBox formControlName="street3" id="address-suffix" placeholder="House No." />
          </div>
          <div class="form-row-no-wrap" *ngIf="busPoBox">
            <input kendoTextBox [value]="'PO Box'" disabled />
            <input kendoTextBox formControlName="street3" placeholder="PO Box Number" />
          </div>
          <div class="error-message" *ngIf="submittedAddress && busPoBox">
            <small *ngIf="addressForm.get('businessAddress.street3')?.hasError('required')">
              PO Box number required.
            </small>
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

      <!-- Button row -->
      <div class="button-row">
        <button kendoButton type="button" class="btn-cancel" (click)="onEdit()">Edit Information</button>
        <button kendoButton type="button" class="btn-cancel" (click)="onCancel()">Cancel</button>
        <button kendoButton type="submit" class="btn-next">Submit</button>
      </div>
    </form>
  </div>
</div>
<app-existing-records [records]="existingNotaries"
                      [visible]="showExistingRecords"
                      [isSubmitting]="isSubmitting"
                      (edit)="onDuplicateEdit()"
                      (ignore)="onDuplicateIgnore()">
</app-existing-records>
add-new-record.component.css file:
kendo-textbox, kendo-datepicker {
  --kendo-color-subtle: #aaa !important;
}

.page-wrapper {
  margin-left: 3rem;
  margin-top: 1rem;
  margin-bottom: 3rem;
}

h2 {
  margin: 0;
  color: #074e72;
}

.first-row {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}

::ng-deep thead {
  background-color: #074e72 !important;
  color: white !important;
}

::ng-deep kendo-datepicker.no-typing input {
    pointer-events: none; /* block typing and focus */
}


.village-hyperlink {
  width: 1.5rem;
  border-radius: 10rem;
  height: 1.4rem;
  background-color: white;
  border-color: #074e72;
  color: #074e72;
}

.zip-inputs {
  display: flex;
  flex-direction: row;
  gap: 0.5rem;
  align-items: baseline;
}

.zip-input-boxes{
    width:4rem;
}

.hyphen{
    color:black;
    font-weight:800;
}

a:visited{
    color:blue;
}

::ng-deep input.k-input:disabled{
    background-color:#cccc!important;
}

.required-indicator {
  display: flex;
  flex-direction: row;
  gap: 0.5rem;
  margin-top: 0.4rem;
/*  justify-content:flex-end;*/
}

.question-asterisk{
    color:red;
    font-size:larger;
    font-weight:900;
}

.asterisk {
  font-size: larger;
  color : red;
  font-weight: bold;
  padding-top:3px;
}

.exception-flex-item{
    display:flex!important;
    flex-direction:row!important;
    gap:0.5rem!important;
}

label {
  font-weight: bold;
}

#salutation {
  width: 5.8rem;
  height: 2.2rem;
}

#suffix {
    width:3.2rem;
}

kendo-datepicker {
    height: 2.2rem;
    width: 12rem;
    margin-right:1rem;
}

.three-size{
    width:3rem;
}

.four-size{
    width:6rem;
}

.dash {
  margin-top: -0.4rem;
  font-size: 2rem;
}

input {
    height:2.2rem;
}

input[type="radio"]{
    height:1rem;
    width:1rem;
    accent-color:#074e72;
}

input::placeholder{
    color: #aaa;
    opacity: 1;
}

.text-danger {
  color: red;
  font-size: larger;
}

.disable-super {
  color:transparent !important;
}

.form-row {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  margin-bottom: 1rem;
}

.flex-item {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.phone-group {
  display: flex;
  gap: 0.5rem;
}

  .phone-group > input {
    width: 4rem;
  }

/* Reserve space for errors so layout doesn’t jump */
.error-message {
  min-height: 1.25rem;
  color: #e00;
  font-size: 0.875rem;
}

.error-message-with-top-marg{
    margin-top:0.1rem;
    font-size: 1rem;
}

.heading-and-error {
  display: flex;
  flex-direction: row;
  gap: 2rem;
}

.button-row {
  display: flex;
  gap: 1rem;
  margin-top: 1rem;
}

.btn-cancel {
  border-color: #074e72;
  background-color: transparent;
  color: #074e72;
  width: 7rem;
  height: 2rem;
}

.btn-next {
  background-color: #074e72;
  border-color: #074e72;
  width: 7rem;
  height: 2rem;
  color: white;
}

.address-section{
    margin-top:1rem;
}

.section-heading{
    color:#074e72;
    font-weight:bold;
    margin-bottom:0.5rem;
}

.label-and-asterisk{
    display: flex;
    flex-direction: row;
    gap: 0.5rem;
}

.label-and-error-message{
    display:flex;
    flex-direction:row;
    gap:2rem;
}

.form-row-no-wrap{
    display: flex;
    gap: 1rem;
    margin-bottom:1rem;
}

#street-number, #address-suffix{
    width: 8rem;
}

kendo-dropdownlist{
    width:10rem;
    height:2.2rem;
    background-color:white!important;
}

.state-and-zip{
    margin-bottom:0!important;
}

.city-town-input-element{
    width:10rem!important;
}

.checkbox-override {
  height: 1rem !important;
  width: 1rem !important;
  margin-top:0.19rem !important
}

::ng-deep input.k-checkbox:checked {
  background-color: white !important;
  color: #074e72 !important;
  border-color: #074e72 !important;
}
styles.css file: 
html, body {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
  box-sizing: border-box;
  font-family: 'Source Sans Pro';
  overflow-x: hidden;
}

/* Override focus states for inputs and widgets */
.k-input:focus,
.k-focus {
  border-color: #074e72 !important;
  box-shadow: 0 0 0 2px rgba(7, 78, 114, 0.2) !important;
}

/* Override DatePicker inner icon and text color */
.k-datepicker .k-icon {
  color: #074e72 !important;
}

/* Override Dropdowns or similar components if needed */
.k-dropdown .k-select {
  color: #074e72 !important;
}
:root {
  --kendo-color-primary: #074e72 !important;
  --kendo-color-primary-hover: #074e72 !important;
  --kendo-color-primary-active: #074e72 !important;
  --kendo-color-primary-on-surface: #074e72 !important;
  --kendo-color-border: #adadad !important;
  --kendo-color-subtle: white !important;
}
