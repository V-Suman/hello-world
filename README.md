Goal: To do a simple lift of the complaints section from the update-notary component and add it to the update-complaint section. 
In more depth: There is an update-notary-details section which contains the add-complaints component. This component allows the user (the one using the application)
to add complaint about other users.. This add complaint flow inside the update-notary-details component contains several important fields like date of complaint, isRemoteNOtary complaint
complaint details, resolution notes, isResolved flag and a hidden resolution date.. datepicker. This is the ADD COMPLAINT FLOW. The general idea is that.. the UPDATE COMPLAINT
flow should be EXACTLY similar to the add-complaints flow.. but with patched values.. since the complaint details are aleady available. 
For example: We are in the notary-profile page.. and when and we expanded the complaint history expansion-panel which has a complaint for the user. Then we go ahead and click on the 
complaint number of a corresponding complaint it navigates us to the update-complaint page. HERE, the flow and the UI of it should be EXACTLY same as the previous add-complaint 
flow.. with all the bells and whistles and validations, and checkbox flag guarding etc etc.. BUT THE CAVEAT HERE IS that it will use the already available component data 
to patch the values such that when the update-complaint page loads.. it loads with values for each of the input items. Remember.. in the update-complaint flow.. there will be 
no way to enable the save button unless the user makes a change to any of the input fields. One more important implementation change from add-complaint flow is that the update-complaint
flow will not have the master checkbox(add complaint checkbox) that the add-complaint flow has. 

Ask me all clarifying questions BEFORE you start implementation. 

notary-profile.component.html file: 
<div *ngIf="profileDetail; else msgToUser" class="page-wrapper">
    <div class="information-section">
        <div class="top-back-link">
            <a (click)="returnToSearch()" class="notary-id-link">
                <<< Return to search results
            </a>
        </div>
        <div class="display-section">
            <div class="notary-detail-information">
                <div class="notary-detail-header-active-complaint">
                    <h3 class="notary-detail-information-header">Notary Detail Information</h3>
                    <div *ngIf="activeComplaints" class="active-complaint-div"> Active Complaint</div>
                </div>
                <div class="flex-item-wrapper">
                    <div class="flex-item">
                        <div class="flex-sub-item">
                            <div class="flex-label">Name</div>
                            <div class="flex-label-data">{{ profileDetail.name }}</div>
                        </div>
                        <div class="flex-sub-item">
                            <div class="flex-label">Residential Address</div>
                            <div class="flex-label-data">{{ profileDetail.residentialAddress }}</div>
                        </div>
                    </div>
                    <div class="flex-item">
                        <div class="flex-sub-item">
                            <div class="flex-label">Date of Birth</div>
                            <div class="flex-label-data">{{ (profileDetail.dateOfBirth  | date:'MM/dd/yyyy') || '---' }}</div>
                        </div>
                        <div class="flex-sub-item">
                            <div class="flex-label">Business Address</div>
                            <div class="flex-label-data">{{ profileDetail.businessAddress }}</div>
                        </div>
                    </div>
                    <div class="flex-item">
                        <div class="flex-sub-item">
                            <div class="flex-label">Email</div>
                            <div class="flex-label-data">{{ profileDetail.emailAddress }}</div>
                        </div>
                        <div class="flex-sub-item">
                            <div class="flex-label">County</div>
                            <div class="flex-label-data">{{ profileDetail.countyName }}</div>
                        </div>
                    </div>
                    <div class="flex-item">
                        <div class="flex-sub-item">
                            <div class="flex-label">Primary Phone</div>
                            <div class="flex-label-data">{{ profileDetail.phone1 }}</div>
                        </div>
                        <div class="flex-sub-item">
                            <div class="flex-label">District</div>
                            <div class="flex-label-data">{{ profileDetail.districtName }}</div>
                        </div>
                    </div>
                    <div class="flex-item">
                        <div class="flex-sub-item">
                            <div class="flex-label">Secondary Phone</div>
                            <div class="flex-label-data">{{ profileDetail.phone2 }}</div>
                        </div>
                        <div class="flex-sub-item">
                            <div class="flex-label">Date of Death</div>
                            <div class="flex-label-data">{{ (profileDetail.dateOfDeath  | date:'MM/dd/yyyy') || '---'}}</div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="notary-account-details">
                <h3 class="notary-account-details-header">Notary Account Details</h3>
                <div class="flex-item-wrapper">
                    <div class="flex-item">
                        <div class="flex-sub-item">
                            <div class="flex-label">Notary ID</div>
                            <div class="flex-label-data">{{ accountDetail?.notaryId }}</div>
                        </div>
                    </div>
                    <div class="flex-item">
                        <div class="flex-sub-item">
                            <div class="flex-label">Status</div>
                            <div class="flex-label-data">{{ accountDetail?.status }}</div>
                        </div>
                    </div>
                    <div class="flex-item">
                        <div class="flex-sub-item">
                            <div class="flex-label">Approval Date</div>
                            <div class="flex-label-data"> {{ (accountDetail?.commissionDate | date:'MM/dd/yyyy') || '---' }}</div>
                        </div>
                    </div>
                    <div class="flex-item">
                        <div class="flex-sub-item">
                            <div class="flex-label">Expiration Date</div>
                            <div class="flex-label-data">{{ (accountDetail?.expirationDate  | date:'MM/dd/yyyy') || '---'}}</div>
                        </div>
                    </div>
                    <div class="flex-item">
                        <div class="flex-sub-item">
                            <div class="flex-label">Has Resigned</div>
                            <div class="flex-label-data">{{ accountDetail?.hasResigned }}</div>
                        </div>
                    </div>
                    <div class="flex-item">
                        <div class="flex-sub-item">
                            <div class="flex-label">Remote Enabled</div>
                            <div class="flex-label-data">{{ accountDetail?.remoteEnabled }}</div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="notary-application-details">
                <app-notary-application-details *ngIf="applicationDetail" [applicationDetail]="applicationDetail"></app-notary-application-details>
            </div>
        </div>
        <div class="grids-section">
            <kendo-expansionpanel title="Notary account history"
                                  [svgExpandIcon]="plusIcon"
                                  [svgCollapseIcon]="minusIcon"
                                  (expand)="onAccountHistoryExpand()">
                <kendo-grid [data]="gridView" [pageable]="true" [skip]="skip" [pageSize]="pageSize" (pageChange)="pageChange($event)">
                    <kendo-grid-column field="accountStatus" title="Account Status"></kendo-grid-column>
                    <kendo-grid-column field="approvalDate" title="Approval Date"></kendo-grid-column>
                    <kendo-grid-column field="expirationDate" title="Expiration Date"></kendo-grid-column>
                    <kendo-grid-column field="resignationDate" title="Resignation Date"></kendo-grid-column>
                    <kendo-grid-column field="paymentDate" title="Payment Date"></kendo-grid-column>
                    <kendo-grid-column field="qualifiedDate" title="Qualified Date"></kendo-grid-column>
                    <kendo-grid-column field="hasResigned" title="Has Resigned"></kendo-grid-column>
                    <kendo-grid-column field="isRemoteEnabled" title="Remote Enabled"></kendo-grid-column>
                </kendo-grid>
            </kendo-expansionpanel>
            <kendo-expansionpanel title="Notes history" [svgExpandIcon]="plusIcon" [svgCollapseIcon]="minusIcon" (expand)="onNotesExpand()">
                <kendo-grid [data]="notesHistory">
                    <kendo-grid-column field="createdOn" title="Date"></kendo-grid-column>
                    <kendo-grid-column field="createdByUser" title="Added by"></kendo-grid-column>
                    <kendo-grid-column field="notes" title="Notes"></kendo-grid-column>
                </kendo-grid>
            </kendo-expansionpanel>
            <kendo-expansionpanel title="Name history" [svgExpandIcon]="plusIcon" [svgCollapseIcon]="minusIcon" (expand)="onNameExpand()">
                <kendo-grid [data]="nameHistory">
                    <kendo-grid-column title="Date">
                        <ng-template kendoGridCellTemplate let-dataItem>
                            {{dataItem.updatedOn !== '' ? (dataItem.updatedOn | date:'MM/dd/yyyy') : '---'}}
                        </ng-template>
                    </kendo-grid-column>
                    <!-- Name column -->
                    <kendo-grid-column title="Name">
                        <ng-template kendoGridCellTemplate let-dataItem>
                            {{ formatFullName(dataItem) }}
                        </ng-template>
                    </kendo-grid-column>

                    <!-- DOB column -->
                    <kendo-grid-column title="DOB">
                        <ng-template kendoGridCellTemplate let-dataItem>
                            {{dataItem.dateOfBirth !== '' ? (dataItem.dateOfBirth | date:'MM/dd/yyyy') : '---'}}
                        </ng-template>
                    </kendo-grid-column>

                    <!-- Type of Change column -->
                    <kendo-grid-column title="Type of Change">
                        <ng-template kendoGridCellTemplate let-dataItem>
                            {{ formatChangeType(dataItem) }}
                        </ng-template>
                    </kendo-grid-column>
                </kendo-grid>
            </kendo-expansionpanel>
            <kendo-expansionpanel title="Address history" [svgExpandIcon]="plusIcon" [svgCollapseIcon]="minusIcon" (expand)="onAddressExpand()">
                <kendo-grid [data]="addressHistory">
                    <kendo-grid-column title="Date">
                        <ng-template kendoGridCellTemplate let-dataItem>
                            {{dataItem.updatedOn !== '' ? (dataItem.updatedOn | date:'MM/dd/yyyy') : '---'}}
                        </ng-template>
                    </kendo-grid-column>

                    <!-- Address -->
                    <kendo-grid-column title="Address">
                        <ng-template kendoGridCellTemplate let-dataItem>
                            {{ formatFullAddress(dataItem) }}
                        </ng-template>
                    </kendo-grid-column>

                    <!-- Address Type -->
                    <kendo-grid-column field="addressType" title="Address Type">
                    </kendo-grid-column>

                    <!-- Type of Change -->
                    <kendo-grid-column title="Type of Change">
                        <ng-template kendoGridCellTemplate let-dataItem>
                            {{ formatAddressChangeType(dataItem) }}
                        </ng-template>
                    </kendo-grid-column>
                </kendo-grid>
            </kendo-expansionpanel>
            <kendo-expansionpanel title="Complaint history" [svgExpandIcon]="plusIcon" [svgCollapseIcon]="minusIcon" (expand)="onComplaintExpand()">
                <kendo-grid [data]="complaintHistory">
                    <kendo-grid-column title="Complaint #">
                        <ng-template kendoGridCellTemplate let-item>
                            <button kendoButton
                                    fillMode="link"
                                    class="notary-id-link complaint-link"
                                    (click)="onComplaintIdClick(item)"
                                    [disabled]="!item?.complaintId || item.complaintId === 0 || !buildEditObject"
                                    [state]="{ buildUpdateComplaintObject:buildUpdateComplaintObject }"
                                    [routerLink]="['/update-complaint', accountDetail?.notaryId]">
                                {{ item?.complaintId ? item.complaintId : '---' }}
                            </button>
                        </ng-template>
                    </kendo-grid-column>

                    <!-- Date of Complaint -->
                    <kendo-grid-column title="Date of Complaint">
                        <ng-template kendoGridCellTemplate let-item>
                            {{item.dateOfComplaint !== '' ? (item.dateOfComplaint | date:'MM/dd/yyyy') : '---'}}
                        </ng-template>
                    </kendo-grid-column>

                    <!-- Complaint Details -->
                    <kendo-grid-column title="Complaint Details">
                        <ng-template kendoGridCellTemplate let-item>
                            {{ item.complaintDetails?.trim() ? item.complaintDetails : '---' }}
                        </ng-template>
                    </kendo-grid-column>

                    <!-- RON Complaint -->
                    <kendo-grid-column title="RON Complaint">
                        <ng-template kendoGridCellTemplate let-item>
                            {{item.isRoncomplaint ? formatYesNo(item.isRoncomplaint) : '---' }}
                        </ng-template>
                    </kendo-grid-column>

                    <!-- Resolved -->
                    <kendo-grid-column title="Resolved">
                        <ng-template kendoGridCellTemplate let-item>
                            {{item.complaintId ? formatYesNo(item.isResolved) : '---'}}
                        </ng-template>
                    </kendo-grid-column>

                    <!-- Resolution Details -->
                    <kendo-grid-column title="Resolution Details">
                        <ng-template kendoGridCellTemplate let-item>
                            {{item.resolutionNotes?.trim() ? item.resolutionNotes : '---' }}
                        </ng-template>
                    </kendo-grid-column>
                </kendo-grid>
            </kendo-expansionpanel>
            <kendo-expansionpanel title="Certificate history"
                                  [svgExpandIcon]="plusIcon"
                                  [svgCollapseIcon]="minusIcon"
                                  (expand)="onCertificateHistoryExpand()">
                <kendo-grid [data]="certificateHistory">
                    <kendo-grid-column field="accountId" title="Account Id"></kendo-grid-column>
                    <kendo-grid-column field="cetificateType" title="Certificate Type"></kendo-grid-column>
                    <kendo-grid-column field="certificateNumber" title="Certificate Number"></kendo-grid-column>
                    <kendo-grid-column field="validationStartDate" title="Validation Start Date"></kendo-grid-column>
                    <kendo-grid-column field="validationEndDate" title="Validation End Date"></kendo-grid-column>
                    <kendo-grid-column field="createdOn" title="Date of Creation"></kendo-grid-column>
                    <kendo-grid-column field="createdByUser" title="Created By"></kendo-grid-column>
                </kendo-grid>
            </kendo-expansionpanel>
        </div>
        <div class="bottom-back-link">
            <a (click)="returnToSearch()" class="notary-id-link">
                <<< Return to search results
            </a>
        </div>
    </div>
    <div class="actions-section">
        <div class="horizontal-line"></div>
        <div class="action-buttons-section">
            <button kendoButton
                    fillMode="clear"
                    class="action-button"
                    [routerLink]="['/update-notary-profile-information', accountDetail?.notaryId]"
                    [state]="{ buildEditObject: buildEditObject}">
                Update profile information
            </button>
            <button kendoButton
                    fillMode="clear"
                    class="action-button"
                    [routerLink]="['/update-notary-details', accountDetail?.notaryId]"
                    [state]="{ buildEditObject: buildEditObject}">
                Update notary details
            </button>
        </div>
    </div>
</div>

<ng-template #msgToUser>
    <p class="information-section" style="color: red;">
        {{messageToUser}}
    </p>
</ng-template>
update-notary-details.component.html:
<div class="page-wrapper">
    <div class="heading-and-asterisk">
        <h2 class="notary-title">
            Update Notary Details - {{ temp }}
        </h2>
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
update-notary-details.component.ts file:
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
        const payload: { notes?: string } = {};
        if (this.addNotesEnabledCtrl.value) {
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
update-complaint.component.html file: 
<div class="page-wrapper">
  <div class="heading-asterisk-complaint">
    <div class="heading-and-asterisk">
      <h2 class="notary-title">
        Update Complaint Details –
        {{ updateComplaintData?.firstName }} {{ updateComplaintData?.lastName }}
        ({{ updateComplaintData?.applicantId }})
      </h2>
      <div class="required-indicator">
        <div class="asterisk">*</div>
        <div class="required-indicator-text"> – Required fields</div>
      </div>
    </div>
    <h3 class="complaint-number-line">
      Complaint - {{ updateComplaintData?.complaintId}}
    </h3>
  </div>
</div>
update-complaint.component.ts file: 
import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';
import { UpdateComplaint } from '../../../models/update-complaint-model/update-complaint.model';

@Component({
  selector: 'app-update-complaint',
  templateUrl: './update-complaint.component.html',
  styleUrls: ['./update-complaint.component.css']
})
export class UpdateComplaintComponent implements OnInit {
  public updateComplaintData?: UpdateComplaint;

  constructor(private router: Router) { }

  ngOnInit(): void {
    const nav = this.router.getCurrentNavigation();
    const fromState = nav?.extras?.state?.['buildUpdateComplaintObject'] as UpdateComplaint | undefined;
    if (fromState) {
      this.updateComplaintData = fromState;
      // Keep the latest for refreshes
      sessionStorage.setItem('updateComplaint', JSON.stringify(fromState));
    } else {
      // Fallback for reloads / direct hits
      const cached = sessionStorage.getItem('updateComplaint');
      if (cached) {
        try {
          this.updateComplaintData = JSON.parse(cached) as UpdateComplaint;
        } catch {
          console.warn('update-complaint: failed to parse cached updateComplaint');
        }
      }
    }
    console.log('update-complaint: received data', this.updateComplaintData);
  }
}
