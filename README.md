Goal: Implement responsiveness based on browser zoom. 
Details:So, the application has users who will need to zoom into the page to view data.
At that point.. I want the grid to be displayed BETWEEN the buttons and expansion panels. 
Normal State 1: 
Expansion panel - 1
Expansion panel - 2
Buttons 
Normal State 1.1 when the user searches for something and grid appears: 
Expansion panel - 1  Results Grid
Expansion panel - 2
Buttons 
Zoomed in State 2:
Expansion panel - 1
Expansion panel - 2
Buttons 
Zoomed in State 2.1 when the user searches for something and grid appears:
Expansion panel - 1 
Expansion panel - 2
Results Grid 
Buttons 

I need this only for this page and Don't worry about the size of the buttons. Leave them 
as is. 

Ask me all clarifying questions before you implement. 
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
                            <div class="flex-label-data">{{ profileDetail.residentialAddress || '---' }}</div>
                        </div>
                    </div>
                    <div class="flex-item">
                        <div class="flex-sub-item">
                            <div class="flex-label">Date of Birth</div>
                            <div class="flex-label-data">{{ (profileDetail.dateOfBirth  | date:'MM/dd/yyyy') || '---' }}</div>
                        </div>
                        <div class="flex-sub-item">
                            <div class="flex-label">Business Address</div>
                            <div class="flex-label-data">{{ profileDetail.businessAddress || '---'  }}</div>
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
                            <div class="flex-label-data">{{ profileDetail.phone2 || '---'}}</div>
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
                            <div class="flex-label-data">{{ accountDetail?.status || '---'  }}</div>
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
                            <div class="flex-label-data">{{ accountDetail?.hasResigned || '---'  }}</div>
                        </div>
                    </div>
                    <div class="flex-item">
                        <div class="flex-sub-item">
                            <div class="flex-label">Remote Enabled</div>
                            <div class="flex-label-data">{{ accountDetail?.remoteEnabled || '---'  }}</div>
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
            <button kendoButton *ngIf="accountDetail?.status != 'Approved'"
                    fillMode="clear"
                    class="action-button"
                    [routerLink]="['/update-notary-app-status', accountDetail?.notaryId]"
                    [state]="{ buildEditObject: buildEditObject}">
                Update application status
            </button>
        </div>
    </div>
</div>

<ng-template #msgToUser>
    <p class="information-section" style="color: red;">
        {{messageToUser}}
    </p>
</ng-template>
notary-profile.component.css file:
.notary-id-link, .notary-id-link:active {
    color: #074e72;
    font-weight: 600;
}

    .notary-id-link:hover {
        text-decoration: underline;
        cursor: pointer;
    }

.page-wrapper {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
}

.information-section {
    display: flex;
    flex-direction: column;
    width: 100%;
    padding: 1rem 1rem 1rem 2rem;
}

::ng-deep .k-button-link-base:focus {
    box-shadow: none !important;
}

.horizontal-line {
    width: 1rem;
    border-left: 2px solid #d5d8db;
}

.actions-section {
    display: flex;
    flex-direction: row;
    width: 16rem;
}

.action-buttons-section {
    display: flex;
    flex-direction: column;
    margin-top: 0.5rem;
}

button:focus {
    border-color: transparent !important;
    background-color: none !important;
}

.action-button {
    color: #074e72;
    font-weight: 900;
    justify-content: unset;
    font-size: 1rem;
}

.grids-section {
    display: flex;
    flex-direction: column;
    gap: 1rem;
    width: 100%;
    margin-bottom: 1rem;
}

.k-expander:not(.k-expanded) {
    border-top-width: 1px !important;
    background-color: #074e72;
}

.k-expander:not(:not(.k-expanded)) {
    border-top-width: 1px !important;
    background-color: #074e72;
}

::ng-deep .k-expander-title {
    color: white !important;
}

::ng-deep .k-expander-content-wrapper {
    background-color: white !important;
}

::ng-deep .k-svg-icon > svg {
    fill: white !important;
}

/*kendo grid related styling from here*/
kendo-grid {
    --kendo-color-border: white !important;
}

::ng-deep .k-column-title {
    color: #074e72;
    font-weight: 600;
}

.display-section {
    display: flex;
    flex-direction: column;
    margin-top: 1rem;
    gap: 1rem;
    margin-bottom: 2rem;
}

.notary-account-details-header, .notary-detail-information-header {
    margin: 0;
    color: #074e72;
}

.notary-detail-information {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.flex-item-wrapper {
    display: flex;
    flex-direction: row;
    column-gap: 1rem;
    justify-content: space-between;
}

.flex-item {
    display: flex;
    flex-direction: column;
    gap: 2rem;
}

.flex-sub-item {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
}

.flex-label {
    font-weight: bold;
}

.notary-account-details {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

::ng-deep .k-grid-header {
    border-bottom: 2px solid #074e72 !important;
}

.notary-detail-header-active-complaint {
    display: flex;
    flex-direction: row;
    column-gap: 1rem;
    align-items: center;
}

.active-complaint-div {
    border: 2px solid red;
    border-radius: 8px;
    padding: 4px;
    font-size: 14px;
    color: red;
}
notary-profile.component.ts file: 
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import mockData from '../../../../mockdata/mock2.json';
import mockProfileData from '../../../../mockdata/mock-profile-data.json';
import expansionData from '../../../../mockdata/mock-expansion-panel-data.json';
import { NotarySearchDataStateService } from '../../../services/helper-services/notary-search-data-state/notary-search-data-state.service';
import { plusIcon, minusIcon, SVGIcon } from "@progress/kendo-svg-icons";
import { NotaryDetailsService } from '../../../services/get-notary-details/notary-details.service';
import { NotaryProfileDetailInfo } from '../../../models/notary-profile-models/notary-profile-detail/notary-profile-detail-info.model';
import { NotaryAccountDetailInfo } from '../../../models/notary-profile-models/notary-account-detail/notary-account-detail-info.model';
import { NotesHistoryService } from '../../../services/get-notes-history/notes-history.service';
import { NameHistory, NoteHistory, AddressHistory, ComplaintHistory, CertificateHistory } from '../../../models/ref-items-model/ref-items.model';
import { NameHistoryService } from '../../../services/get-name-history/name-history.service';
import { AddressHistoryService } from '../../../services/get-address-history/address-history.service';
import { ComplaintHistoryService } from '../../../services/get-complaint-history/complaint-history.service';
import { NotaryApplicationDetails } from '../../../models/notary-profile-models/notary-application-detail/notary-application-details.model';
import { AccountHistoryService } from '../../../services/get-account-history/account-history.service';
import { GridDataResult, PageChangeEvent } from '@progress/kendo-angular-grid';
import { CertificateHistoryService } from '../../../services/get-certificate-history/certificate-history.service';
import { BuildEditObject, UpdatePersonalProfileInformation } from '../../../models/update-profile-info-model/update-profile-info.model';
import { UpdateComplaint } from '../../../models/update-complaint-model/update-complaint.model';

@Component({
    selector: 'app-notary-profile',
    templateUrl: './notary-profile.component.html',
    styleUrls: ['./notary-profile.component.css']
})
export class NotaryProfileComponent implements OnInit {

    public notaryId!: number;
    public plusIcon: SVGIcon = plusIcon;
    public minusIcon: SVGIcon = minusIcon;

    public profileDetail?: NotaryProfileDetailInfo;
    public accountDetail?: NotaryAccountDetailInfo;
    public notesHistory: NoteHistory[] = [];
    private notesHistoryLoaded = false;
    public nameHistory: NameHistory[] = [];
    private nameHistoryLoaded = false;
    public addressHistory: AddressHistory[] = [];
    private addressHistoryLoaded = false;
    public complaintHistory: ComplaintHistory[] = [];
    private complaintHistoryLoaded = false;
    public applicationDetail?: NotaryApplicationDetails;
    public accountHistory: Array<{
        accountId: string | null;
        accountStatus: string;
        approvalDate: string;
        expirationDate: string;
        resignationDate: string;
        paymentDate: string;
        qualifiedDate: string;
        hasResigned: string;
        isRemoteEnabled: string;
    }> = [];
    public accountHistoryLoaded = false;
    public gridView!: GridDataResult;
    public pageSize = 5;
    public skip = 0;
    public certificateHistory: Array<{
        accountId: number | string;
        cetificateType: string;
        certificateNumber: string;
        validationStartDate: string;
        validationEndDate: string;
        createdOn: string;
        createdByUser: string;
    }> = [];
    public certificateHistoryLoaded = false;
    public buildEditObject!: BuildEditObject;
    public buildUpdateComplaintObject!: UpdateComplaint;
    public activeComplaints: boolean = false;

    messageToUser: string = "Searching for Notary ID ";
    constructor(private route: ActivatedRoute,
        private searchStateData: NotarySearchDataStateService,
        private router: Router,
        private notaryDetails: NotaryDetailsService,
        private notesHistoryService: NotesHistoryService,
        private nameHistoryService: NameHistoryService,
        private addressHistoryService: AddressHistoryService,
        private accountHistoryService: AccountHistoryService,
        private certificateHistoryService: CertificateHistoryService,
        private complaintHistoryService: ComplaintHistoryService) { }

    ngOnInit(): void {
        const idParam = this.route.snapshot.paramMap.get('id');
        if (!idParam) {
            console.error('No "id" found in route parameters!');
            this.messageToUser = "No notary ID provided for profile search";
            return;
        }

        this.notaryId = Number(idParam);
        if (isNaN(this.notaryId)) {
            console.error(`Route parameter "id" is not a valid number: ${idParam}`);
            this.messageToUser = "No valid notary ID provided for profile search";
            return;
        }
        this.messageToUser = this.messageToUser + idParam;

        this.notaryDetails.getNotaryProfile(Number(idParam)).subscribe({
            next: response => {
                this.profileDetail = this.mapToDetailInfo(response);
                this.accountDetail = this.mapToAccountDetailInfo(response);
                this.applicationDetail = this.mapToApplicationDetailInfo(response);
                this.buildEditObject = this.mapToEditObject(response);
                console.log(this.buildEditObject);
                console.log(response);
            },
            error: err => {
                this.messageToUser = "There was an error finding notary profile with ID " + idParam;
                console.error('Error fetching notary profile:', err);
            }
        });

    }

    private buildUpdateComplaintFor(item: ComplaintHistory): UpdateComplaint {
        const pid = this.buildEditObject?.personalInfoDetails;
        if (!pid) {
            throw new Error('buildEditObject.personalInfoDetails is not ready yet.');
        }

        const accountId = pid.accountId;

        return {
            // person/account fields
            firstName: pid.firstName,
            middleName: pid.middleName,
            lastName: pid.lastName,
            applicantId: pid.applicantId,
            accountId: pid.accountId,

            // complaint-specific fields (raw values)
            complaintId: item.complaintId,
            dateOfComplaint: item.dateOfComplaint,
            isRoncomplaint: item.isRoncomplaint,
            complaintDetails: item.complaintDetails,
            isResolved: item.isResolved,
            resolutionNotes: item.resolutionNotes,
            resolutionDate: item.resolutionDate
        };
    }

    public onComplaintIdClick(item: ComplaintHistory): void {
        try {
            sessionStorage.removeItem('updateComplaint');
            this.buildUpdateComplaintObject = this.buildUpdateComplaintFor(item);

            // Persist for reloads
            sessionStorage.setItem(
                'updateComplaint',
                JSON.stringify(this.buildUpdateComplaintObject)
            );
        } catch (e) {
            console.error(e);
        }
    }

    private mapToEditObject(resp: any): BuildEditObject {
        if (resp.accountDetails == null) {
            const pid = resp.personalInfoDetails;
            const email = resp.emailAddress;
            return {
                personalInfoDetails: {
                    accountId: 0,
                    emailAddress: email,
                    salutationTypeId: pid.salutationTypeId,
                    salutationType: pid.salutationType,
                    firstName: pid.firstName,
                    middleName: pid.middleName,
                    lastName: pid.lastName,
                    suffix: pid.suffix,
                    dateOfBirth: pid.dateOfBirth,
                    applicantId: pid.applicantId,
                    notaryIdentifier: pid.notaryIdentifier,
                    dateOfDeath: pid.dateOfDeath,
                    dateOfResignation: null,
                    addressDetails: pid.addressDetails.map((a: any) => ({
                        addressId: a.addressId,
                        applicantId: a.applicantId,
                        addressTypeId: a.addressTypeId,
                        addressType: a.addressType,
                        isPrefered: a.isPrefered,
                        isPoBox: a.isPoBox,
                        streetNumber: a.streetNumber,
                        streetName: a.streetName,
                        aptNumber: a.aptNumber,
                        addressLine2: a.addressLine2,
                        zipCode: a.zipCode,
                        zipPlus: a.zipPlus,
                        city: a.city,
                        county: a.county,
                        district: a.district,
                        stateId: a.stateId,
                        state: a.state,
                    })),
                    contactDetails: pid.contactDetails.map((c: any) => ({
                        contactTypeId: c.contactTypeId,
                        contactType: c.contactType,
                        contactId: c.contactId,
                        applicantId: c.applicantId,
                        contactValue: c.contactValue,
                        isPrimary: c.isPrimary,
                    })),
                }
            };
        } else {
            const pid = resp.personalInfoDetails;
            const email = resp.emailAddress;
            const accountIdentifier = resp.accountDetails?.accountId;
            const acctDetails = resp.accountDetails;
            return {
                personalInfoDetails: {
                    accountId: accountIdentifier,
                    emailAddress: email,
                    salutationTypeId: pid.salutationTypeId,
                    salutationType: pid.salutationType,
                    firstName: pid.firstName,
                    middleName: pid.middleName,
                    lastName: pid.lastName,
                    suffix: pid.suffix,
                    dateOfBirth: pid.dateOfBirth,
                    applicantId: pid.applicantId,
                    notaryIdentifier: pid.notaryIdentifier,
                    dateOfDeath: pid.dateOfDeath,
                    dateOfResignation: acctDetails.resignationDate,
                    addressDetails: pid.addressDetails.map((a: any) => ({
                        addressId: a.addressId,
                        applicantId: a.applicantId,
                        addressTypeId: a.addressTypeId,
                        addressType: a.addressType,
                        isPrefered: a.isPrefered,
                        isPoBox: a.isPoBox,
                        streetNumber: a.streetNumber,
                        streetName: a.streetName,
                        aptNumber: a.aptNumber,
                        addressLine2: a.addressLine2,
                        zipCode: a.zipCode,
                        zipPlus: a.zipPlus,
                        city: a.city,
                        county: a.county,
                        district: a.district,
                        stateId: a.stateId,
                        state: a.state,
                    })),
                    contactDetails: pid.contactDetails.map((c: any) => ({
                        contactTypeId: c.contactTypeId,
                        contactType: c.contactType,
                        contactId: c.contactId,
                        applicantId: c.applicantId,
                        contactValue: c.contactValue,
                        isPrimary: c.isPrimary,
                    })),
                }
            };
        }
    }

    private mapToDetailInfo(resp: any): NotaryProfileDetailInfo {
        const pid = resp.personalInfoDetails;
        this.activeComplaints = resp.hasActiveComplaints;
        const addrs = pid.addressDetails as any[];
        const contacts = pid.contactDetails as any[];

        const name = [pid.firstName, pid.middleName, pid.lastName, pid.suffix]
            .filter(n => !!n)
            .join(' ');

        const dateOfBirth = pid.dateOfBirth.split('T')[0];
        const dateOfDeath = pid.dateOfDeath ? pid.dateOfDeath.split('T')[0] : '';

        const emailAddress = resp.emailAddress != null ? resp.emailAddress : '---';


        const phoneContacts = (contacts || []).filter((c: any) =>
            c?.contactTypeId === 1 || (c?.contactType || '').toLowerCase() === 'phone'
        );

        const primaryPhone = phoneContacts.find((p: any) => !!p.isPrimary)?.contactValue || '';
        const secondaryPhone = phoneContacts.find((p: any) => !p.isPrimary)?.contactValue || '';

        const phone1 = primaryPhone || null;
        const phone2 = secondaryPhone || null;


        const preferred = addrs.find((a: any) => a.isPrefered);
        const countyName = preferred?.county ?? '---';
        const districtName = preferred?.district ?? '---';


        const formatAddress = (a: any) => {
            let parts: string = `${a.streetNumber} ${a.streetName}`;
            if (a.aptNumber) parts += ` ${a.aptNumber}`;
            if (a.addressLine2) parts += ` ${a.addressLine2}`;
            parts += `, ${a.city}, ${a.state} ${a.zipCode}`;
            if (a.zipPlus) parts += `-${a.zipPlus}`;
            return parts;
        };

        const resAddrObj = addrs.find((a: any) => a.addressType === 'Residential');
        const businessAddrObj = addrs.find((a: any) => a.addressType === 'Business');

        const residentialAddress = resAddrObj ? formatAddress(resAddrObj) : '';
        const businessAddress = businessAddrObj ? formatAddress(businessAddrObj) : '---';

        return {
            name,
            dateOfBirth,
            emailAddress,
            phone1,
            phone2,
            residentialAddress,
            businessAddress,
            countyName,
            districtName,
            dateOfDeath
        };
    }

    private mapToAccountDetailInfo(resp: any): NotaryAccountDetailInfo {
        if (resp.accountDetails == null) {
            console.log("inside null");
            const persInfo = resp.personalInfoDetails;
            console.log(persInfo);
            const acct = resp.applicationDetails;
            return {
                notaryId: persInfo.applicantId,
                status: null,
                commissionDate: acct.approvalDate ? acct.approvalDate.split('T')[0] : null,
                expirationDate: null,
                resignationDate: null,
                hasResigned: null,
                remoteEnabled: null
            };
        }
        else {
            console.log("inside non-null");
            const acct = resp.accountDetails;
            return {
                notaryId: acct.applicantId,
                status: acct.accountStatus,
                commissionDate: acct.approvalDate.split('T')[0],
                expirationDate: acct.expirationDate.split('T')[0],
                resignationDate: acct.resignationDate ? acct.resignationDate.split('T')[0] : null,
                hasResigned: acct.hasResigned ? 'Yes' : 'No',
                remoteEnabled: acct.isRemoteNotary ? 'Yes' : 'No'
            };
        }
    }

    public onNotesExpand(): void {
        if (this.notesHistoryLoaded) {
            return;
        }

        this.notesHistoryService.getNotesHistory(this.notaryId).subscribe({
            next: hist => {
                if (hist.length === 0) {
                    // no history show one row of ‘---’
                    this.notesHistory = [{
                        noteId: 0,
                        accountId: this.notaryId,
                        notes: '---',
                        createdOn: '---',
                        createdByUser: '---'
                    }];
                } else {
                    this.notesHistory = hist;
                }
                this.notesHistoryLoaded = true;
            },
            error: err => {
                console.error('Error fetching notes history:', err);
                // optionally populate the empty-state row on error as well
                this.notesHistory = [{
                    noteId: 0,
                    accountId: this.notaryId,
                    notes: '---',
                    createdOn: '---',
                    createdByUser: '---'
                }];
                this.notesHistoryLoaded = true;
            }
        });
    }

    public formatFullName(item: NameHistory): string {
        const parts = [item.firstName, item.middleName, item.lastName]
            .filter(n => !!n && n.trim() !== '');
        return parts.length ? parts.join(' ') : '---';
    }

    /** Returns 'Official' if isOfficalNameChange, 'Correction' if isNameCorrection, else '---' */
    public formatChangeType(item: NameHistory): string {
        if (item.isOfficialNameChange) {
            return 'Official';
        }
        if (item.isNameCorrection) {
            return 'Correction';
        }
        return '---';
    }

    public onNameExpand(): void {
        if (this.nameHistoryLoaded) { return; }

        this.nameHistoryService.getNameHistory(this.notaryId).subscribe({
            next: (data: NameHistory[]) => {
                if (!data || data.length === 0) {
                    // placeholder row of all '---'
                    this.nameHistory = [{
                        updatedOn: '',
                        firstName: '---',
                        middleName: '',
                        lastName: '',
                        dateOfBirth: '',
                        isOfficialNameChange: false,
                        isNameCorrection: false
                    }];
                } else {
                    this.nameHistory = data;
                }
                this.nameHistoryLoaded = true;
            },
            error: err => {
                console.error('Error fetching name history:', err);
                this.nameHistory = [{
                    updatedOn: '',
                    firstName: '---',
                    middleName: '',
                    lastName: '',
                    dateOfBirth: '',
                    isOfficialNameChange: false,
                    isNameCorrection: false
                }];
                this.nameHistoryLoaded = true;
            }
        });
    }

    public formatFullAddress(item: AddressHistory): string {
        const parts: string[] = [];
        if (item.streetNumber) parts.push(item.streetNumber);
        if (item.streetName) parts.push(item.streetName);
        if (item.aptNumber) parts.push(item.aptNumber as string);
        if (item.addressLine2) parts.push(item.addressLine2 as string);
        if (item.city) parts.push(item.city);

        if (item.zipPlus != null && `${item.zipPlus}`.trim() !== '') {
            if (item.state) parts.push(item.state);
            parts.push(`${item.zipCode}-${item.zipPlus}`);
        } else if (item.state) {
            parts.push(`${item.state} - ${item.zipCode}`);
        }

        return parts.join(', ');
    }

    /** Official vs Correction vs --- */
    public formatAddressChangeType(item: AddressHistory): string {
        if (item.isOfficalAddressChange) return 'Official';
        if (item.isAddressCorrection) return 'Correction';
        return '';
    }

    public onAddressExpand(): void {
        if (this.addressHistoryLoaded) { return; }

        this.addressHistoryService.getAddressHistory(this.notaryId).subscribe({
            next: (data: AddressHistory[]) => {
                if (!data || data.length === 0) {
                    // one row of all '---'
                    this.addressHistory = [{
                        updatedOn: '',
                        streetNumber: '',
                        streetName: '---',
                        aptNumber: '',
                        addressLine2: '',
                        city: '',
                        state: '',
                        zipCode: '',
                        zipPlus: '',
                        addressType: '---',
                        isOfficalAddressChange: false,
                        isAddressCorrection: false
                    }];
                } else {
                    this.addressHistory = data;
                }
                this.addressHistoryLoaded = true;
            },
            error: err => {
                console.error('Error fetching address history:', err);
                this.addressHistory = [{
                    updatedOn: '',
                    streetNumber: '---',
                    streetName: '---',
                    aptNumber: '---',
                    addressLine2: '---',
                    city: '---',
                    state: '---',
                    zipCode: '---',
                    zipPlus: '',
                    addressType: '---',
                    isOfficalAddressChange: false,
                    isAddressCorrection: false
                }];
                this.addressHistoryLoaded = true;
            }
        });
    }

    public formatYesNo(value: boolean | string | null | undefined): string {
        if (value === null || value === undefined || `${value}`.trim() === '') {
            return '---';
        }
        if (typeof value === 'boolean') return value ? 'Yes' : 'No';
        const v = (value as string).toLowerCase();
        if (v === 'true') return 'Yes';
        if (v === 'false') return 'No';
        return '---';
    }

    public onComplaintExpand(): void {
        if (this.complaintHistoryLoaded) { return; }

        this.complaintHistoryService.getComplaintHistory(this.notaryId)
            .subscribe({
                next: (data: ComplaintHistory[]) => {
                    if (!data || data.length === 0) {
                        this.complaintHistory = [{
                            complaintId: 0,
                            dateOfComplaint: '',
                            complaintDetails: '---',
                            isRoncomplaint: '',
                            isResolved: false,
                            resolutionNotes: '---',
                            resolutionDate: '---'
                        }];
                    } else {
                        this.complaintHistory = data;
                    }
                    this.complaintHistoryLoaded = true;
                },
                error: err => {
                    console.error('Error fetching complaint history:', err);
                    this.complaintHistory = [{
                        complaintId: 0,
                        dateOfComplaint: '',
                        complaintDetails: '---',
                        isRoncomplaint: '',
                        isResolved: false,
                        resolutionNotes: '---',
                        resolutionDate: '---'
                    }];
                    this.complaintHistoryLoaded = true;
                }
            });
    }

    private formatDate(dateStr?: string | null): string {
        if (!dateStr) {
            return '';
        }
        // split off any time portion, then MM/DD/YYYY
        const [datePart] = dateStr.split('T');
        const [year, month, day] = datePart.split('-');
        return `${month}/${day}/${year}`;
    }

    private mapToApplicationDetailInfo(resp: any): NotaryApplicationDetails {
        const app = resp.applicationDetails || {};

        return {
            applicationDate: this.formatDate(app.applicationDate),
            applicationStatus: app.applicationStatus ?? '---',
            applicationStatusDate: this.formatDate(app.applicationStatusDate),
            approvalDate: this.formatDate(app.approvalDate),
            dueDate: this.formatDate(app.dueDate),
            applicationNextStep: app.applicationNextStep == '' ? '---' : app.applicationNextStep,
            applicationStatusToolTip: app.applicationStatusToolTip ?? '---'
        };
    }

    public onAccountHistoryExpand(): void {
        if (this.accountHistoryLoaded) return;

        this.accountHistoryService.getAccountHistory(this.notaryId).subscribe({
            next: raw => {
                const data = Array.isArray(raw) ? raw : [];

                // keep only items with a real accountId
                const usable = data.filter(item => item && item.accountId != null);
                console.log(usable);

                if (usable.length === 0) {
                    // either empty array OR only null-accountId rows single placeholder row
                    this.accountHistory = [{
                        accountId: null,
                        accountStatus: '---',
                        approvalDate: '---',
                        expirationDate: '---',
                        resignationDate: '---',
                        paymentDate: '---',
                        qualifiedDate: '---',
                        hasResigned: '---',
                        isRemoteEnabled: '---'
                    }];
                } else {
                    // map as usual, no sorting, preserve API order
                    this.accountHistory = usable.map(item => ({
                        accountId: null,
                        accountStatus: item.accountStatus?.trim() ? item.accountStatus : '---',
                        approvalDate: this.formatDate(item.approvalDate),       
                        expirationDate: this.formatDate(item.expirationDate),
                        resignationDate: this.formatDate(item.resignationDate),
                        paymentDate: this.formatDate(item.paymentDate),
                        qualifiedDate: this.formatDate(item.qualifiedDate),
                        hasResigned: this.formatYesNo(item.hasResigned),         
                        isRemoteEnabled: this.formatYesNo(item.isRemoteEnabled), 
                    }));
                }

                this.accountHistoryLoaded = true;
                this.loadGrid(); 
            },
            error: err => {
                console.error('Error fetching account history:', err);
                this.accountHistory = [{
                    accountId: null,
                    accountStatus: '---',
                    approvalDate: '',
                    expirationDate: '',
                    resignationDate: '',
                    paymentDate: '',
                    qualifiedDate: '',
                    hasResigned: '---',
                    isRemoteEnabled: '---'
                }];
                this.accountHistoryLoaded = true;
                this.loadGrid();
            }
        });
    }

    private loadGrid(): void {
        this.gridView = {
            data: this.accountHistory.slice(this.skip, this.skip + this.pageSize),
            total: this.accountHistory.length
        };
    }

    public pageChange(event: PageChangeEvent): void {
        this.skip = event.skip;
        this.loadGrid();
    }

    public onCertificateHistoryExpand(): void {
        if (this.certificateHistoryLoaded) {
            return;
        }

        this.certificateHistoryService
            .getAddressHistory(this.notaryId)   // (yes, the method is named getAddressHistory)
            .subscribe({
                next: data => {
                    if (!data || data.length === 0) {
                        // one all-‘---’ placeholder row
                        this.certificateHistory = [{
                            accountId: '---',
                            cetificateType: '---',
                            certificateNumber: '---',
                            validationStartDate: '---',
                            validationEndDate: '---',
                            createdOn: '---',
                            createdByUser: '---'
                        }];
                    } else {
                        this.certificateHistory = data.map(item => ({
                            accountId: item.accountId,
                            cetificateType: item.cetificateType?.trim() ? item.cetificateType : '---',
                            certificateNumber: item.certificateNumber?.trim() ? item.certificateNumber : '---',
                            validationStartDate: this.formatDate(item.validationStartDate),
                            validationEndDate: this.formatDate(item.validationEndDate),
                            createdOn: this.formatDate(item.createdOn),
                            createdByUser: item.createdByUser?.trim() ? item.createdByUser : '---',
                        }));
                    }
                    this.certificateHistoryLoaded = true;
                },
                error: err => {
                    console.error('Error fetching certificate history:', err);
                    this.certificateHistoryLoaded = true;
                    this.certificateHistory = [{
                        accountId: '---',
                        cetificateType: '---',
                        certificateNumber: '---',
                        validationStartDate: '',
                        validationEndDate: '',
                        createdOn: '',
                        createdByUser: '---'
                    }];
                }
            });
    }


    public returnToSearch(): void {
        this.router.navigate(['/notary-records']);
    }
}
