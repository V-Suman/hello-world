Goal: To edit the functionality of the Get County & District Button 
Details: I added 3 new items to the existing add-new-record flow which are the county name, district name 
and a button called Get County & District. Now.. ideally I should be binding the Get County & District button
to some (click) handler for it to do anything. But.. if I click the button.. suddenly the form validators are firing 
Fix this issue. Form validators should not be fired. Furthermore.. if I click on the Get County and District 
button belonging to business address the Form validators for residence are getting fired. LIke what? 

Also.. it throws 5 of the same error in the console saying "Error: The `kendo-formfield` component should contain only one control of type NgControl with a formControlName"

Ask me clarifying questions before you implement. Fix these things else I will bludgeon you with a jackhammer

Html file: 
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
                            valueField="value"
                            [class.is-placeholder]="!form.get('salutation')?.value">
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
                          [max]="maxDob"
                          [class.is-placeholder]="!form.get('dateOfBirth')?.value"
                          placeholder="MM/DD/YYYY">
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
        <input kendoTextBox id="email" formControlName="email" placeholder="example@abc.com" maxlength="100"/>
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
            Primary Phone Number is required.
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
                         placeholder="Zip"
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
                         placeholder="Zip + 4"
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

          <div class="form-row-no-wrap">
              <kendo-formfield class="flex-item">
                  <label class="label-and-asterisk" kendoLabel>
                      County Name
                      <sup class="text-danger" [class.disable-super]="preferred==='Business'">*</sup>
                  </label>
                  <input kendoTextBox id="county-name" placeholder="County Name" />
              </kendo-formfield>
              <kendo-formfield class="flex-item">
                  <label class="label-and-asterisk" kendoLabel>
                      District Name
                      <sup class="text-danger" [class.disable-super]="preferred==='Business'">*</sup>
                  </label>
                  <input kendoTextBox id="district-name" placeholder="District Name" />
              </kendo-formfield>
              <kendo-formfield class="flex-item">
                  <label class="label-and-asterisk" kendoLabel>
                      <sup class="text-danger disable-super">*</sup>
                  </label>
                  <button kendoButton class="btn-next width-modifier">Get County & District</button>
              </kendo-formfield>
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
                                     placeholder="Zip"
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
                                     placeholder="Zip + 4"
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

              <div class="form-row-no-wrap">
                  <kendo-formfield class="flex-item">
                      <label class="label-and-asterisk" kendoLabel>
                          County Name
                          <sup class="text-danger" [class.disable-super]="preferred==='Residence'">*</sup>
                      </label>
                      <input kendoTextBox id="county-name" placeholder="County Name" />
                  </kendo-formfield>
                  <kendo-formfield class="flex-item">
                      <label class="label-and-asterisk" kendoLabel>
                          District Name
                          <sup class="text-danger" [class.disable-super]="preferred==='Residence'">*</sup>
                      </label>
                      <input kendoTextBox id="district-name" placeholder="District Name" />
                  </kendo-formfield>
                  <kendo-formfield class="flex-item">
                      <label class="label-and-asterisk" kendoLabel>
                          <sup class="text-danger disable-super">*</sup>
                      </label>
                      <button kendoButton class="btn-next width-modifier">Get County & District</button>
                  </kendo-formfield>
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
ts file: 
import { Component, OnInit } from '@angular/core';
import {
  FormBuilder,
  FormGroup,
  FormControl,
  Validators,
  AbstractControl,
  ValidationErrors,
  ValidatorFn
} from '@angular/forms';
import { RefGetterService } from '../../../services/helper-services/ref-get-service/ref-getter.service';
import { CityCountyRef, SalutationRef, StateRef } from '../../../models/ref-items-model/ref-items.model';
import { AddNewRecordRequest, AddressDto, ContactDto, PersonalInfo } from '../../../models/add-new-record-model/add-new-record-request.model';
import { take } from 'rxjs/operators';
import { AddNotaryResponse, NewNotaryRecordService } from '../../../services/new-record/new-notary-record.service';
import { NotarySearchService } from '../../../services/search/notary-search.service';
import { GeneralSearchRequest, LiveSearchWrapper } from '../../../models/live-search-model/live-search.model';
import { Router } from '@angular/router';
import { minAgeValidator } from '../../../reactive-validators/min-age.validator';

@Component({
  selector: 'app-add-new-record',
  templateUrl: './add-new-record.component.html',
  styleUrls: ['./add-new-record.component.css']
})
export class AddNewRecordComponent implements OnInit {
  public form!: FormGroup;
  public addressForm!: FormGroup;
  public submitted = false;
  public submittedAddress = false;
  public stateOptions: string[] = [];
  //public cityOptions: string[] = [];
  public cityOptions: CityCountyRef[] = [];
  //public countyValue: string = '';
  public salutationOptions: string[] = [];
  maxDate: Date = new Date();
  // Only letters, spaces, hyphens or slashes
  private namePattern = /^[A-Za-z\s\-\/]+$/;

  private suffixPattern = /^[A-Za-z0-9\s\-\/\.]+$/;;

  //to call once and cache
  private allCityCountyData: CityCountyRef[] = [];

  public salutations: SalutationRef[] = [];
  public stateRefs: StateRef[] = [];

  public existingNotaries: any;
  public showExistingRecords: boolean = false;
  private pendingRequest: AddNewRecordRequest | null = null;
  public isSubmitting = false;
    minDob: Date = new Date(1800, 0, 1); // Jan 1, 1800
    maxDob: Date = this.computeMaxDob(); // today minus 18 years

    private computeMaxDob(): Date {
        const today = new Date();
        return new Date(today.getFullYear() - 18, today.getMonth(), today.getDate());
    }

  showAddressSection: boolean = false;
  viewHeading: string = 'Personal Information'

  constructor(private fb: FormBuilder,
              private refGetter: RefGetterService,
              private newNotaryService: NewNotaryRecordService,
              private router: Router,
      private notarySearch: NotarySearchService) {
      const today = new Date();
      this.maxDate = new Date(today.getFullYear() - 18, today.getMonth(), today.getDate());
  }

  ngOnInit(): void {
    this.buildPersonalForm();
    this.buildAddressForm();
    this.getSalutationsForDropDown();
    this.getStatesForDropDown();
    this.cacheCityCountyData();

    // watch the two state controls:
    const resState = this.addressForm.get('residenceAddress.state')!;
    const busState = this.addressForm.get('businessAddress.state')!;

    resState.valueChanges.subscribe(() => {
      const resAddressGroup = this.addressForm.get('residenceAddress');
      resAddressGroup?.get('cityTown')!.reset();
      this.updateCityOptions();
    });

    busState.valueChanges.subscribe(() => {
      const busAddressGroup = this.addressForm.get('businessAddress');
      busAddressGroup?.get('cityTown')!.reset();
      this.updateCityOptions();
    });
  }

  private getStatesForDropDown(): void {
    this.refGetter.getStates().subscribe(
      (all: StateRef[]) => {
        this.stateRefs = all;
        this.stateOptions = all.map(s => s.value);
      },
      err => console.error('Failed to load states', err)
    );
  }

  private getSalutationsForDropDown(): void {
    this.refGetter.getSalutations().subscribe(
      (all: SalutationRef[]) => {
        this.salutations = all;
        this.salutationOptions = all.map(x => x.value);
      },
      err => console.error('Failed to load salutations', err)
    );
  }

  private cacheCityCountyData(): void {
    this.refGetter.getCityCounty().pipe(take(1)).subscribe(
      (all: CityCountyRef[]) => {
        this.allCityCountyData = all;
      },
      err => console.error('Failed to load city/county', err)
    );
  }

  private updateCityOptions() {
    const resStateValue = this.addressForm.get('residenceAddress.state')!.value;
    const busStateValue = this.addressForm.get('businessAddress.state')!.value;

    if (resStateValue === 'Massachusetts' || busStateValue === 'Massachusetts') {
      this.cityOptions = this.allCityCountyData;
    } else {
      this.cityOptions = []; // Clear the list only if NEITHER is Massachusetts
    }
  }

  public get preferred(): 'Residence' | 'Business' {
    return this.addressForm.get('preferredAddress')!.value;
  }

  private buildPersonalForm() {
    this.form = this.fb.group({
      salutation: ['', Validators.required],
      firstName: ['', [Validators.required, Validators.pattern(this.namePattern)]],
      middleName: ['', Validators.pattern(this.namePattern)],
      lastName: ['', [Validators.required, Validators.pattern(this.namePattern)]],
      suffix: ['', Validators.pattern(this.suffixPattern)],
      dateOfBirth: [null, [minAgeValidator(18), Validators.required]],
      email: ['', [Validators.required, Validators.email]],
      primaryPhone1: ['', [Validators.required, Validators.pattern(/^\d{3}$/)]],
      primaryPhone2: ['', [Validators.required, Validators.pattern(/^\d{3}$/)]],
      primaryPhone3: ['', [Validators.required, Validators.pattern(/^\d{4}$/)]],
      secondaryPhone1: ['', Validators.pattern(/^\d{3}$/)],
      secondaryPhone2: ['', Validators.pattern(/^\d{3}$/)],
      secondaryPhone3: ['', Validators.pattern(/^\d{4}$/)],
    });
  }

  private buildAddressForm() {
    this.addressForm = this.fb.group({
      preferredAddress: ['Residence', Validators.required],

      // Residence sub‐group
      residenceAddress: this.fb.group({
        street1: [''],  // required: at least one of street1/2/3
        street2: [''],
        street3: [''],
        street4: [''],
        state: [''],
        isPoBox: [false],
        zipCode: ['', [Validators.required, Validators.pattern(/^\d{5}$/)]],
        zipPlus: ['', Validators.pattern(/^$|^\d{2,4}$/)],
        cityTown: ['']
      }, { validators: this.streetValidator() }),

      // Business sub‐group
      businessAddress: this.fb.group({
        street1: [''],
        street2: [''],
        street3: [''],
        street4: [''],
        state: [''],
        isPoBox: [false],
        zipCode: ['', [Validators.required, Validators.pattern(/^\d{5}$/)]],
        zipPlus: ['', Validators.pattern(/^$|^\d{2,4}$/)],
        cityTown: ['']
      }, { validators: this.streetValidator() })
    });

    const busGroup = this.addressForm.get('businessAddress') as FormGroup;
    busGroup.get('isPoBox')!
      .valueChanges
      .subscribe(isPo => this.onBusinessPoBoxToggle(isPo));

    // whenever the radio changes, re‐apply validators
    this.addressForm.get('preferredAddress')!
      .valueChanges
      .subscribe(pref => this.updateAddressValidators(pref));
    // initialize validators for Residence default
    this.updateAddressValidators('Residence');
  }

  get busPoBox() {
    return this.addressForm.get('businessAddress.isPoBox')!.value;
  }

  private onBusinessPoBoxToggle(isPoBox: boolean) {
    const bus = this.addressForm.get('businessAddress') as FormGroup;
    const s1 = bus.get('street1')!;
    const s2 = bus.get('street2')!;
    const s3 = bus.get('street3')!;

    if (isPoBox) {
      s1.reset(); s2.reset();
      s1.disable(); s2.disable();
      s3.setValidators([Validators.required]);
    } else {
      s1.enable(); s2.enable();
      s3.clearValidators();
    }
    [s1, s2, s3].forEach(c => c.updateValueAndValidity({ onlySelf: true }));
    bus.clearValidators();
    if (isPoBox) {
      bus.setValidators(this.optionalGroupValidator('businessAddress'));
    } else {
      bus.setValidators(this.streetValidator());
    }
    bus.updateValueAndValidity({ onlySelf: true });
  }

  private streetValidator(): ValidatorFn {
    return (group: AbstractControl): ValidationErrors | null => {
      const g = group as FormGroup;
      const s1 = g.get('street1')!.value;
      const s2 = g.get('street2')!.value;
      return (!s1 && !s2) ? { noStreet: true } : null;
    };
    }

    openDatePicker(picker: any) {
        picker.toggle(true);// picker.open();   // now it resolves the method correctly
    }

    private optionalGroupValidator(prefix: 'residenceAddress' | 'businessAddress'): ValidatorFn {
        return (group: AbstractControl): ValidationErrors | null => {
            const g = group as FormGroup;
            const isPo = g.get('isPoBox')?.value === true;
            const hasAny = Object.entries(g.value)
                .some(([k, v]) => k !== 'isPoBox' && typeof v === 'string' && v.trim().length > 0);

            if (!hasAny) return null; 

            if (isPo) {
                const poNum = (g.get('street3')?.value ?? '').toString().trim();
                if (!poNum) {

                    g.get('street3')?.setErrors({ required: true });
                    return { poBoxNumberReq: true };
                }
            } else {

                const s1 = (g.get('street1')?.value ?? '').toString().trim();
                const s2 = (g.get('street2')?.value ?? '').toString().trim();
                if (!s1 && !s2) return { noStreet: true };
            }

            for (const field of ['state', 'zipCode', 'cityTown'] as const) {
                const ctrl = g.get(field);
                const val = (ctrl?.value ?? '').toString().trim();
                if (!val) {
                    ctrl?.setErrors({ required: true });
                    return { [`${field}Req`]: true } as any;
                }
            }

            return null;
        };
    }

  private updateAddressValidators(pref: 'Residence' | 'Business') {
    const resGroup = this.addressForm.get('residenceAddress') as FormGroup;
    const busGroup = this.addressForm.get('businessAddress') as FormGroup;

    for (const grp of [resGroup, busGroup]) {
      // clear group‐level validators
      grp.clearValidators();
      Object.values(grp.controls).forEach(ctrl => {
        ctrl.clearValidators();
        // re‐run validation so old errors go away
        ctrl.updateValueAndValidity({ onlySelf: true, emitEvent: false });
      });
    }

    if (pref === 'Residence') {
      resGroup.setValidators(this.streetValidator());
      resGroup.updateValueAndValidity({ onlySelf: true, emitEvent: false });

      ['state', 'zipCode', 'cityTown'].forEach(name => {
        const c = resGroup.get(name)!;
        if (name === 'zipCode') {
          // required AND exactly 5 digits
          c.setValidators([Validators.required, Validators.pattern(/^\d{5}$/)]);
        } else if(name === 'cityTown') { 
            c.setValidators([Validators.required, Validators.pattern(/^[A-Za-z0-9 '.,\/;:\-#&]+$/)]);
        } else {
          c.setValidators(Validators.required);
        }
        c.updateValueAndValidity({ onlySelf: true, emitEvent: false });
      });

      // re-apply pattern to zipPlus (optional but must be 2–4 digits if entered)
      const zp = resGroup.get('zipPlus')!;
      zp.setValidators(Validators.pattern(/^$|^\d{2,4}$/));
      zp.updateValueAndValidity({ onlySelf: true, emitEvent: false });

      // 2b) Business is fully optional (only validate if user types something)
      busGroup.setValidators(this.optionalGroupValidator('businessAddress'));
      busGroup.updateValueAndValidity({ onlySelf: true, emitEvent: false });
    }
    else {
      // 3a) Business is required:
      busGroup.setValidators(this.streetValidator());
      busGroup.updateValueAndValidity({ onlySelf: true, emitEvent: false });

      ['state', 'zipCode', 'cityTown'].forEach(name => {
        const c = busGroup.get(name)!;
        if (name === 'zipCode') {
          c.setValidators([Validators.required, Validators.pattern(/^\d{5}$/)]);
        } else if(name === 'cityTown') { 
            c.setValidators([Validators.required, Validators.pattern(/^[A-Za-z0-9 '.,\/;:\-#&]+$/)]);
        } else {
          c.setValidators(Validators.required);
        }
        c.updateValueAndValidity({ onlySelf: true, emitEvent: false });
      });

      const zpBus = busGroup.get('zipPlus')!;
      zpBus.setValidators(Validators.pattern(/^$|^\d{2,4}$/));
      zpBus.updateValueAndValidity({ onlySelf: true, emitEvent: false });

      // 3b) Residence is optional
      resGroup.setValidators(this.optionalGroupValidator('residenceAddress'));
      resGroup.updateValueAndValidity({ onlySelf: true, emitEvent: false });
    }
  }

  public onSubmit(): void {
    this.submitted = true;
    this.form.markAllAsTouched();
    if (this.form.invalid) {return;}
    this.triggerAddressSection();
  }

  public onAddressSubmit(): void {
    this.submittedAddress = true;
    this.addressForm.markAllAsTouched();
    if (this.addressForm.invalid) { return; }

    // helper: turn '' or all-whitespace into null
    const nullOr = (s: string | undefined | null): string | null =>
      s?.trim() ? s.trim() : null;

    const pv = this.form.value;         // personal form values
    const av = this.addressForm.value;  // address form values
    const cityTown = av.preferredAddress === 'Business'
                   ? av.businessAddress.cityTown
                   : av.residenceAddress.cityTown;

    // 1) build ContactDto[]
    const primaryPhone = pv.primaryPhone1 + pv.primaryPhone2 + pv.primaryPhone3;
    const secondaryPhone = (pv.secondaryPhone1 && pv.secondaryPhone2 && pv.secondaryPhone3)
      ? pv.secondaryPhone1 + pv.secondaryPhone2 + pv.secondaryPhone3
      : null;
    const contacts: ContactDto[] = [
      { contactTypeId: 1, contactValue: primaryPhone, isPrimary: true },
      ...(secondaryPhone
        ? [{ contactTypeId: 1, contactValue: secondaryPhone, isPrimary: false }]
        : []),
      { contactTypeId: 2, contactValue: pv.email, isPrimary: false }
    ];

    // 2) build AddressDto[]
      const hasGroupData = (grp: any, ignoreKeys: string[] = ['isPoBox']) =>
          Object.entries(grp).some(([k, v]) =>
              !ignoreKeys.includes(k) &&
              typeof v === 'string' &&
              v.trim().length > 0
          );

      const buildAddress = (grp: any, typeId: number, preferred: boolean): AddressDto => {
          const isPo = grp.isPoBox === true;

          const stateName = grp.state as string;
          const stateRef = this.stateRefs.find(s => s.value === stateName);
          const stateId = stateRef?.stateId ?? 0;

          const poNumber = (grp.street3 ?? '').toString().trim();
          const streetName = isPo
              ? (poNumber ? `PO Box ${poNumber}` : 'PO Box')
              : (grp.street2?.trim() || null);

          return {
              addressTypeId: typeId,
              isPrefered: preferred,
              isPoBox: isPo,
              streetNumber: isPo ? null : (grp.street1?.trim() || null),
              streetName,
              aptNumber: isPo ? null : (grp.street3?.trim() || null),
              addressLine2: grp.street4?.trim() || null,
              zipCode: grp.zipCode,                 
              zipPlus: (grp.zipPlus?.trim() || null),
              city: grp.cityTown,                  
              stateId
          };
      };

      console.log(buildAddress);

      const addresses: AddressDto[] = [];
      const pref = av.preferredAddress as 'Residence' | 'Business';

      // Always include the preferred address (validators ensure it's complete)
      if (pref === 'Residence') {
          addresses.push(buildAddress(av.residenceAddress, 1, true));
          // Optionally include Business if the user typed anything there
          if (hasGroupData(av.businessAddress)) {
              addresses.push(buildAddress(av.businessAddress, 2, false));
          }
      } else { // pref === 'Business'
          addresses.push(buildAddress(av.businessAddress, 2, true));
          // Optionally include Residence if the user typed anything there
          if (hasGroupData(av.residenceAddress)) {
              addresses.push(buildAddress(av.residenceAddress, 1, false));
          }
      }

    // 3) map salutation ID
    const salId = this.salutations
      .find(s => s.value === pv.salutation)!
      .salutationTypeId;

    // 4) assemble PersonalInfo payload
    const personalInfo: PersonalInfo = {
      salutationTypeId: salId,
      firstName: pv.firstName,
      middleName: nullOr(pv.middleName),
      lastName: pv.lastName,
      suffix: nullOr(pv.suffix),
      dateOfBirth: this.formatDate(pv.dateOfBirth),
      contactsDto: contacts,
      addressDto: addresses
    };

    // 5) final request
    const request: AddNewRecordRequest = { personalInfoDto: personalInfo };
    this.pendingRequest = request;
    console.log(request);
    const gs: GeneralSearchRequest = {
      applicantId: null,
      firstName: this.form.value.firstName,
      lastName: this.form.value.lastName,
      cityTown: cityTown || '',
      approvalDate: null,
      dateOfBirth: this.formatDate(this.form.value.dateOfBirth),
      remoteNotaryOnly: false
    };
    const searchWrapper: LiveSearchWrapper = { generalSearch: gs };

    this.notarySearch.searchNotaries(searchWrapper)
      .subscribe(
        res => {
          const msg = res.message || '';

          if (msg.includes('Below are the 0 record(s) that match your search criteria')) {
              // 1) no existing match: go ahead and add
              //console.log(request);
            this.newNotaryService.addNewNotaryRecord(request)
              .subscribe(
                addRes => {
                  const rawCode: string = addRes.code || '';
                  const parts = rawCode.split(':');
                  if (parts.length > 1) {
                    const idStr = parts[1].trim();
                    const applicantId = Number(idStr);
                    if (!isNaN(applicantId)) {
                      // 3) navigate with the numeric ID
                      this.router.navigate(['/notary-profile', applicantId]);
                    } else {
                      console.error('Unable to parse applicantId from code:', rawCode);
                      alert('Unexpected response format; cannot navigate to profile.');
                      this.router.navigate(['/']);
                    }
                  } else {
                    console.error('Unexpected code format:', rawCode);
                    alert('Unexpected response format; cannot navigate to profile.');
                    this.router.navigate(['/']);
                  }
                },
                addErr => {
                  console.error('Failed to add notary record', addErr);
                  alert('There was an issue trying to create the new record.');
                  this.router.navigate(['/']);
                }
              );
          } else {
            // found one or more matches—log them and skip creation
            this.existingNotaries = res.notarySearchResultsInternalDto;
            console.log(this.existingNotaries);
            this.showExistingRecords = true;
          }
        },
        searchErr => {
          console.error('Notary search failed', searchErr);
          alert('There was an issue searching for existing records.');
        }
      );
  }

  triggerAddressSection(): void {
    this.showAddressSection = true;
    this.viewHeading = 'Address Information';
  }

  private formatDate(input: Date | string | null): string {
    if (!input) {
      return '';
    }
    // if it’s already a Date use it, otherwise coerce
    const d = input instanceof Date ? input : new Date(input);
    if (isNaN(d.getTime())) {
      return '';
    }
    const y = d.getFullYear();
    const m = String(d.getMonth() + 1).padStart(2, '0');
    const day = String(d.getDate()).padStart(2, '0');
    return `${y}-${m}-${day}`;
  }

  public onCancel(): void {
    // e.g. navigate back or to another route
    window.history.back();
  }

  public onPhoneInput(
    curr: HTMLInputElement,
    next?: HTMLInputElement
  ): void {
    if (next && curr.value.length === curr.maxLength) {
      next.focus();
    }
  }

  public onPhoneKeydown(
    evt: KeyboardEvent,
    curr: HTMLInputElement,
    prev: HTMLInputElement | null
  ): void {
    if (evt.key === 'Backspace' && curr.value.length === 0 && prev) {
      evt.preventDefault();
      prev.focus();
      // place caret at end
      const len = prev.value.length;
      prev.setSelectionRange(len, len);
    }
  }

  public onPhonePaste(
    evt: ClipboardEvent,
    inputs: HTMLInputElement[]
  ): void {
    evt.preventDefault();
    const text = evt.clipboardData
      ?.getData('text/plain')
      .replace(/\D/g, '') || '';
    let rest = text;

    inputs.forEach((inp) => {
      const ml = inp.maxLength;
      const chunk = rest.slice(0, ml);
      rest = rest.slice(ml);

      inp.value = chunk;
      // update the FormControl too
      const name = inp.getAttribute('formControlName');
      if (name) {
        this.form.get(name)!.setValue(chunk);
      }
    });

    // focus the first not-full box, or the last one if all are full
    const next = inputs.find(i => i.value.length < i.maxLength) || inputs[inputs.length - 1];
    next.focus();
    next.setSelectionRange(next.value.length, next.value.length);
  }

  public allowOnlyNumbers(event: KeyboardEvent): void {
    // Allow navigation keys (backspace, tab, arrows):
    if (event.key.length === 1 && !/^\d$/.test(event.key)) {
      event.preventDefault();
    }
  }

  public onZipPaste(event: ClipboardEvent, controlPath: string): void {
    event.preventDefault();
    const text = event.clipboardData?.getData('text/plain') || '';
    const digits = text.replace(/\D/g, '');
    const ctrl = this.addressForm.get(controlPath);
    if (ctrl) {
      ctrl.setValue(digits);
    }
  }

  public onEdit(): void {
    this.showAddressSection = false;
  }

  public onDuplicateEdit(): void {
    this.showExistingRecords = false;
    this.showAddressSection = false;
  }

  public onDuplicateIgnore(): void {
    if (!this.pendingRequest) { return; }

    this.isSubmitting = true;
    this.newNotaryService.addNewNotaryRecord(this.pendingRequest)
      .subscribe(
        addRes => this.handleAddSuccess(addRes),
        addErr => {
          console.error('Failed to add notary record', addErr);
          this.isSubmitting = false;
          alert('There was an issue trying to create the new record.');
          this.router.navigate(['/']);
        }
      );
  }
  private createAndNavigate(request: AddNewRecordRequest) {
    this.isSubmitting = true;
    this.newNotaryService.addNewNotaryRecord(request)
      .subscribe(
        addRes => this.handleAddSuccess(addRes),
        addErr => {
          console.error('Failed to add notary record', addErr);
          this.isSubmitting = false;
          alert('There was an issue trying to create the new record.');
          this.router.navigate(['/']);
        }
      );
  }

  /** parse `addRes.code`, pull the numeric ID, navigate  */
  private handleAddSuccess(addRes: any) {
    const rawCode = addRes.code || '';
    const parts = rawCode.split(':');
    if (parts.length > 1) {
      const id = Number(parts[1].trim());
      if (!isNaN(id)) {
        this.router.navigate(['/notary-profile', id]);
        return;
      }
    }
    // fallback on any unexpected format
    console.error('Unexpected code format:', rawCode);
    alert('Unexpected response format; cannot navigate to profile.');
    this.router.navigate(['/']);
  }

  public testDupeMatchDialog(): void {
    this.showExistingRecords = true;
    this.existingNotaries = [
      {
        "applicantId": 1460,
        "approvalDate": "2004-11-10",
        "cityTown": "Attleboro",
        "county": "Bristol",
        "createdDate": "2004-10-28",
        "dateOfBirth": "1974-07-31",
        "firstName": "Jennifer",
        "isRemoteNotary": false,
        "lastName": "Savini",
        "middleName": "Lee",
        "newRenewal": "Renew"
      }
    ];
  }
}
