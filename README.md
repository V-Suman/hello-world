Goal: To resolve a bug.. wherein downstream flows are getting effected by not properly processing (or idk what is the issue) applicantId field. 
More clearly: if a user creates a new record via the add-new-record flow.. and we route the user to internal-profile page 
using the applicantId (that we recive in the code section of the response for add-new-record).. the applicantId is appearing on the url as null 
for the downstream flows and the downstream update-personal-information and update-notary-details flow which HEAVILY depend on the 
applicantId are getting disturbed as well. Essentially.. any new record I create I am not able to do update-personal-information beacuse 
when I land on that page.. the form IS broken. 
My suspicions: I suspect that in my notary-profile.component.html I have<button kendoButton
                    fillMode="clear"
                    class="action-button"
                    [routerLink]="['/update-notary-profile-information', accountDetail?.notaryId]"
                    [state]="{ buildEditObject: buildEditObject}">
                Update profile information
            </button>
which has the accountDetail?.notaryId where ? is a null coaelescing operator. But, if I remove the ? operator from my HTML code and run 
the app again it throws error saying Object is possibly be undefined.
The thing I don't understand: Why is this working for already existing records where I navigate to the notary-profile page via the notary-records 
view.. and everyting works as expected. 

Ask me any clarifying questions you have before you suggest changes. 

notary-profile.component.html: 
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
notary-profile.component.ts:
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

            //console.log('buildUpdateComplaintObject:', this.buildUpdateComplaintObject);
        } catch (e) {
            console.error(e);
        }
    }

    private mapToEditObject(resp: any): BuildEditObject {
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

    private mapToDetailInfo(resp: any): NotaryProfileDetailInfo {
        const pid = resp.personalInfoDetails;
        this.activeComplaints = resp.hasActiveComplaints;
        const addrs = pid.addressDetails as any[];
        const phones = pid.contactDetails as any[];

        // 1) Name
        const name = [pid.firstName, pid.middleName, pid.lastName, pid.suffix]
            .filter(n => !!n)
            .join(' ');

        // 2 & 10) Dates
        const dateOfBirth = pid.dateOfBirth.split('T')[0];
        const dateOfDeath = pid.dateOfDeath
            ? pid.dateOfDeath.split('T')[0]
            : '';

        // 3) Email
        const emailAddress = resp.emailAddress != null ? resp.emailAddress : '---';

        // 4 & 5) Phones

        const primaryPhone = phones.length > 0
            ? phones.find(p => p.isPrimary)?.contactValue
            : "";

        const secondary = phones.length > 0
            ? phones.find(p => !p.isPrimary)
            : ""
        const phone1 = primaryPhone;
        const phone2 = secondary ? secondary.contactValue : null;

        // Preferred address for county & district
        const preferred = addrs.find(a => a.isPrefered)!;
        const countyName = preferred.county;
        const districtName = preferred.district;

        // Helper to format an address object
        const formatAddress = (a: any) => {
            let parts: string = "";
            parts = a.streetNumber + " " + a.streetName;
            
            if (a.aptNumber !== "")
                parts = parts + " " + a.aptNumber;

            if (a.addressLine2 != null)
                parts = parts + " " + a.addressLine2;
            parts = parts + ", ";

            parts = parts + a.city + ", " + a.state + " " + a.zipCode;
            if (a.zipPlus != null) {
                parts = parts + "-" + a.zipPlus;
                
            }
            return parts;
        };

        // 6) Residential
        const resAddrObj = addrs.find(a => a.addressType === 'Residential');
        const residentialAddress = resAddrObj
            ? formatAddress(resAddrObj)
            : '';

        // 7) Business
        const busAddrObj = addrs.find(a => a.addressType === 'Business');
        const businessAddress = busAddrObj
            ? formatAddress(busAddrObj)
            : '---';

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
                        createdOn: '',
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
                    createdOn: '',
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

    public formatYesNo(value: boolean | string): string {
        if (typeof value === 'boolean') {
            return value ? 'Yes' : 'No';
        }
        // treat string 'true'/'false'
        return value === 'true' ? 'Yes' : 'No';
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
        if (this.accountHistoryLoaded) {
            return;
        }

        this.accountHistoryService.getAccountHistory(this.notaryId).subscribe({
            next: data => {
                if (!data || data.length === 0) {
                    this.accountHistory = [{
                        accountStatus: '---',
                        approvalDate: '',
                        expirationDate: '',
                        resignationDate: '',
                        paymentDate: '',
                        qualifiedDate: '',
                        hasResigned: '---',
                        isRemoteEnabled: '---'
                    }];
                } else {
                    this.accountHistory = data.map(item => ({
                        // string fields: blank/null  ‘---’
                        accountStatus: item.accountStatus?.trim() ? item.accountStatus : '---',
                        // all dates  MM/DD/YYYY
                        approvalDate: this.formatDate(item.approvalDate),
                        expirationDate: this.formatDate(item.expirationDate),
                        resignationDate: this.formatDate(item.resignationDate),
                        paymentDate: this.formatDate(item.paymentDate),
                        qualifiedDate: this.formatDate(item.qualifiedDate),
                        // booleans  Yes/No
                        hasResigned: this.formatYesNo(item.hasResigned),
                        isRemoteEnabled: this.formatYesNo(item.isRemoteEnabled),
                    }));
                }
                this.accountHistoryLoaded = true;
                this.loadGrid();
            },
            error: err => {
                console.error('Error fetching account history:', err);
                // on error, same “all ---” row
                this.accountHistory = [{
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
add-new-record.component.ts file: 
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

  showAddressSection: boolean = false;
  viewHeading: string = 'Personal Information'

  constructor(private fb: FormBuilder,
              private refGetter: RefGetterService,
              private newNotaryService: NewNotaryRecordService,
              private router: Router,
              private notarySearch: NotarySearchService) { }

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
      dateOfBirth: [null, Validators.required],
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

  private optionalGroupValidator(prefix: 'residenceAddress' | 'businessAddress'): ValidatorFn {
    return (group: AbstractControl): ValidationErrors | null => {
      const g = group as FormGroup;
      const vals = Object.values(g.value).some(v => !!v);
      if (!vals) {
        return null; // completely empty is OK
      }
      // now enforce: streetValidator + required on state/zipCode/cityTown
      const streetErr = this.streetValidator()(g);
      if (streetErr) {
        return { noStreet: true };
      }
      for (const field of ['state', 'zipCode', 'cityTown']) {
        if (!g.get(field)!.value) {
          return { [`${field}Req`]: true };
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
    const makeAddress = (grp: any, typeId: number, preferred: boolean): AddressDto => {
      const isPo = grp.isPoBox === true;
      const stateName = grp.state;
      const stateRef = this.stateRefs.find(s => s.value === stateName);
      if (!stateRef) {
        console.error('Unknow Statet')
      }
      const stateId = stateRef?.stateId ?? 0;
      return {
        addressTypeId: typeId,
        isPrefered: preferred,
        isPoBox: isPo,
        streetNumber: isPo ? null : nullOr(grp.street1),
        streetName: isPo ? 'PO Box' : nullOr(grp.street2),
        aptNumber: nullOr(grp.street3),
        addressLine2: nullOr(grp.street4),
        zipCode: grp.zipCode,
        zipPlus: nullOr(grp.zipPlus),
        city: grp.cityTown,
        stateId: stateId
      };
    };

    // always include the preferred “Residence” address
    const addresses: AddressDto[] = [
      makeAddress(
        av.residenceAddress,
        1,
        av.preferredAddress === 'Residence'
      )
    ];

    // check if the user actually entered any BUSINESS fields (ignore the isPoBox boolean)
    const { isPoBox, ...restBusiness } = av.businessAddress;
    const hasBusinessData = Object
      .values(restBusiness)
      .some(val =>
        typeof val === 'string' &&
        val.trim().length > 0
      );

    if (hasBusinessData) {
      addresses.push(
        makeAddress(
          av.businessAddress,
          2,
          av.preferredAddress === 'Business'
        )
      );
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
The notary-records flow using which it is working currently notary-records.component.html:
<div class="page-wrapper">
  <div class="instructions-and-search-wrapper">
    <h2 class="notary-title">Find Notary Record</h2>
    <!-- Simple Focussed Search -->
    <kendo-expansionpanel title="Notary search" [expanded]="isNotaryExpanded" (expand)="handleNotaryExpand()" (collapse)="handleNotaryCollapse()">
      <form class="notary-form">
        <!-- Row 1: First name, Last name and Remote Notaries Only -->
        <div class="form-row">
          <kendo-formfield>
            <label for="firstName">First Name</label>
            <kendo-textbox id="firstName"
                           [(ngModel)]="firstName"
                           name="firstName"
                           placeholder="John"
                           class="custom-input">
            </kendo-textbox>
          </kendo-formfield>

          <kendo-formfield>
            <label for="lastName">Last Name</label>
            <kendo-textbox id="lastName"
                           [(ngModel)]="lastName"
                           name="lastName"
                           placeholder="Appleseed"
                           class="custom-input">
            </kendo-textbox>
          </kendo-formfield>

          <kendo-formfield>
            <label for="cityTown">City / Town</label>
            <kendo-textbox id="cityTown"
                           [(ngModel)]="cityTown"
                           name="cityTown"
                           placeholder="Amherst"
                           class="custom-input">
            </kendo-textbox>
          </kendo-formfield>

        </div>

        <!-- Row 2: City/Town and Approval Date -->
        <div class="form-row">
          <div class="approval-range-group">
            <kendo-formfield>
              <label for="fromApprovalDate">Approval Date</label>
              <kendo-datepicker id="fromApprovalDate"
                                [(ngModel)]="fromApprovalDate"
                                name="fromApprovalDate"
                                class="custom-input"
                                placeholder="MM/DD/YYYY">
              </kendo-datepicker>
            </kendo-formfield>
            <kendo-formfield>
              <label for="dateOfBirth">Date of birth</label>
              <kendo-datepicker id="dateOfBirth"
                                [(ngModel)]="dateOfBirth"
                                name="dateOfBirth"
                                class="custom-input"
                                placeholder="MM/DD/YYYY">
              </kendo-datepicker>
            </kendo-formfield>
          </div>
        </div>

        <!-- Row 3: Notary ID -->
        <div class="form-row">
          <kendo-formfield>
            <label for="notaryId">Notary ID #</label>
            <kendo-textbox id="notaryId"
                           [(ngModel)]="notaryId"
                           name="notaryId"
                           placeholder="123456"
                           class="custom-input"
                           
                           (keypress)="allowOnlyNumbers($event)"
                           (input)="onNotaryIdInput($event)"
                           (blur)="validateNotaryId()">
            </kendo-textbox>
            <div *ngIf="notaryIdError" class="error-message">{{ notaryIdError }}</div>
          </kendo-formfield>
          <kendo-formfield>
            <div class="notary-category-group">
              <label for="remoteNotariesOnly" class="remoteNotariesLabel">Filter Remote Notaries Only</label>
              <kendo-checkbox id="remoteNotariesOnly"
                              [(ngModel)]="remoteNotariesOnly"
                              name="remoteNotariesOnly">
              </kendo-checkbox>
            </div>
          </kendo-formfield>
        </div>
      </form>
    </kendo-expansionpanel>
    <!-- Advanced Date Range Search -->
    <kendo-expansionpanel title="Date range search" [expanded]="isDateRangeExpanded" (expand)="handleDateRangeExpand()" (collapse)="handleDateRangeCollapse()">
      <div class="approval-range-group date-range-special">
        <div class="date-range-and-dash">
          <kendo-formfield>
            <label for="fromApprovalDate">Approval Date From</label>
            <kendo-datepicker id="fromApprovalDate"
                              [(ngModel)]="fromApprovalDate"
                              name="fromApprovalDate"
                              class="custom-input"
                              placeholder="MM/DD/YYYY">
            </kendo-datepicker>
          </kendo-formfield>
          <span class="dash">-</span>
          <kendo-formfield>
            <label for="fromApprovalDate">Approval Date To</label>
            <kendo-datepicker id="toApprovalDate"
                              [(ngModel)]="toApprovalDate"
                              [disabled]="!fromApprovalDate"
                              name="toApprovalDate"
                              class="custom-input"
                              placeholder="MM/DD/YYYY">
            </kendo-datepicker>
          </kendo-formfield>
        </div>
        <div class="type-options">
          <kendo-formfield>
            <label>New</label>
            <kendo-radiobutton name="recordType"
                               [(ngModel)]="recordType"
                               id="type-new"
                               value="1">
            </kendo-radiobutton>
          </kendo-formfield>

          <kendo-formfield>
            <label>Renewal</label>
            <kendo-radiobutton name="recordType"
                               [(ngModel)]="recordType"
                               id="type-renewed"
                               value="2">
            </kendo-radiobutton>
          </kendo-formfield>

          <kendo-formfield>
            <label>Both</label>
            <kendo-radiobutton name="recordType"
                               [(ngModel)]="recordType"
                               id="type-both"
                               value="3">
            </kendo-radiobutton>
          </kendo-formfield>
        </div>
      </div>
    </kendo-expansionpanel>
    <div class="form-row button-row">
      <button kendoButton
              (click)="onSearch()"
              [primary]="true"
              class="search-button"
              [disabled]="isFormEmpty()">
        Search
      </button>
      <button kendoButton (click)="onClear()" class="custom-button-alt">
        Clear
      </button>
      <button kendoButton (click)="onReset()" class="custom-button-alt">
        Reset
      </button>
      <button *ngIf="toggleSearchGrid" kendoButton routerLink="/notary-records/add-new-record" class="custom-button-alt">
        Add New Record
      </button>
    </div>
  </div>

  <!-- Search Results Grid -->
  <div *ngIf="toggleSearchGrid" class="search-results-grid">
    <div class="matched-records-text">
      {{displayMessage}}
    </div>

    <kendo-grid [kendoGridBinding]="searchResults"
                [pageSize]="pageSize"
                [style.height]="enableSimple ? null : (searchResults.length > 0 ? '30.69rem' : null)"
                [skip]="skip"
                [pageable]="enableSimple"
                [sortable]="{ allowUnsort: true, mode: 'multiple' }"
                [reorderable]="true"
                (pageChange)="pageChange($event)"
                [scrollable]="enableSimple ? 'none' : 'scrollable'"
                class="custom-grid">
      <!-- Columns -->
      <kendo-grid-column field="applicantId" title="Notary ID">
        <ng-template kendoGridCellTemplate let-dataItem>
          <a [routerLink]="['/notary-profile', dataItem.applicantId]" class="notary-id-link">
            {{ dataItem.applicantId }}
          </a>
        </ng-template>
      </kendo-grid-column>
      <kendo-grid-column field="newRenewal" title="New/Renew"></kendo-grid-column>
      <kendo-grid-column field="lastName" title="Last Name"></kendo-grid-column>
      <kendo-grid-column field="firstName" title="First Name" [sortable]="true"></kendo-grid-column>
      <kendo-grid-column field="middleName" title="Middle Name"></kendo-grid-column>
      <kendo-grid-column field="cityTown" title="City / Town"></kendo-grid-column>
      <kendo-grid-column field="dateOfBirth" title="Date of Birth">
        <ng-template kendoGridCellTemplate let-dataItem>
          {{ dataItem.dateOfBirth | date:'MM/dd/yyyy' }}
        </ng-template>
      </kendo-grid-column>
      <kendo-grid-column field="county" title="County"></kendo-grid-column>
      <kendo-grid-column field="approvalDate" title="Approval Date">
        <ng-template kendoGridCellTemplate let-dataItem>
          {{ dataItem.approvalDate | date:'MM/dd/yyyy' }}
        </ng-template>
      </kendo-grid-column>
      <kendo-grid-column field="createdDate" title="Created Date">
        <ng-template kendoGridCellTemplate let-dataItem>
          {{ dataItem.createdDate  | date:'MM/dd/yyyy' }}
        </ng-template>
      </kendo-grid-column>
      <!--No Created Date Available-->
      <kendo-grid-column field="isRemoteNotary" title="RON Status">
        <ng-template kendoGridCellTemplate let-dataItem>
          {{ dataItem.isRemoteNotary ? 'Yes' : 'No' }}
        </ng-template>
      </kendo-grid-column>
    </kendo-grid>
  </div>
</div>
notary-records.component.ts file this flow also workss but sharing just in case:
import { Component, OnInit, ViewChild } from '@angular/core';
import { NotarySearchService } from '../../../services/search/notary-search.service';
import { PageChangeEvent } from '@progress/kendo-angular-grid';
import { Router } from '@angular/router';
import { NotarySearchDataStateService } from '../../../services/helper-services/notary-search-data-state/notary-search-data-state.service';
import { LiveSearchWrapper } from '../../../models/live-search-model/live-search.model';

@Component({
  selector: 'app-notary-records',
  templateUrl: './notary-records.component.html',
  styleUrls: ['./notary-records.component.css']
})
export class NotaryRecordsComponent implements OnInit {
  notaryId: string = '';
  cityTown: string = '';
  remoteNotariesOnly: boolean = false;
  firstName: string = '';
  lastName: string = '';
  // For Approval Date Range – we are using two date pickers.
  fromApprovalDate: Date | null = null;
  toApprovalDate: Date | null = null;

  // Validation error
  public notaryIdError: string = '';

  // Properties for storing the response data
  displayMessage: string = '';
  searchResults: any[] = [];

  // Grid properties
  toggleSearchGrid: boolean = false;
  public pageSize = 10;
  public skip = 0;

  //Accordion enabling
  enableSimple: boolean = true;
  enableDateRange: boolean = false;

  //Record types
  public recordType: "1"|"2"|"3" = "3";

  //expansion panel
  expandedPanel: boolean = true;
  public isNotaryExpanded = true;
  public isDateRangeExpanded = false;

  //min and max dates for datepickers
  public max!: Date;
  public min!: Date;

  //to check if simple fields were cleared
  public hasClearedSimpleFields = false;

  public dateOfBirth: Date | null = null;

  constructor(private search: NotarySearchService, private router: Router, private searchStateData: NotarySearchDataStateService) {
    const today = new Date();

    this.max = new Date(
      today.getFullYear() - 18,
      today.getMonth(),
      today.getDate()
    );
    this.min = new Date(
      today.getFullYear() - 80,
      today.getMonth(),
      today.getDate()
    );
  }

  ngOnInit(): void {
    this.checkForRehydration();
  }

  private checkForRehydration(): void {
    const crit = this.searchStateData.lastCriteria;
    if (!crit) { return; }

    //getting back the panels to prev state
    if (crit.generalSearch) {
      const g = crit.generalSearch;
      this.notaryId = g.applicantId != null ? g.applicantId.toString() : '';
      this.firstName = g.firstName;
      this.lastName = g.lastName;
      this.cityTown = g.cityTown;
      this.remoteNotariesOnly = g.remoteNotaryOnly;

      this.fromApprovalDate = g.approvalDate ? this.parseLocalDate(g.approvalDate) : null;

      this.enableSimple = true;
      this.enableDateRange = false;
      this.isNotaryExpanded = true;
      this.isDateRangeExpanded = false;
    }
    else if (crit.dateRangeSearch) {
      const d = crit.dateRangeSearch;
      this.fromApprovalDate = d.approvalDateFrom ? this.parseLocalDate(d.approvalDateFrom) : null;
      this.toApprovalDate = d.approvalDateTo ? this.parseLocalDate(d.approvalDateTo) : null;
      //this.recordType = d.searchType as "1"|"2"|"3";

      this.enableSimple = false;
      this.enableDateRange = true;
      this.isNotaryExpanded = false;
      this.isDateRangeExpanded = true;
    }

    //make a fresh new call
    this.fetchResults(crit);
  }

  /** to hit the api */
  private fetchResults(wrapper: LiveSearchWrapper): void {
    //general or daterange
    if (wrapper.generalSearch) {
      this.search.searchNotaries(wrapper).subscribe({
        next: (response) => {
          this.applyResults(response);
        },
        error: (err) => console.error('Re-fetch general search failed', err)
      });
    }
    else if (wrapper.dateRangeSearch) {
      this.search.searchNotariesWithDateRange(wrapper).subscribe({
        next: (response) => {
          this.applyResults(response);
        },
        error: (err) => console.error('Re-fetch date-range failed', err)
      });
    }
  }

  /** Shared logic for applying and persisting results */
  private applyResults(response: { message: string; notarySearchResultsInternalDto: any[] }): void {
    this.searchResults = response.notarySearchResultsInternalDto.map(item => ({
      ...item,
      approvalDate: item.approvalDate ? this.parseLocalDate(item.approvalDate) : null,
      createdDate: item.createdDate ? this.parseLocalDate(item.createdDate) : null
    }));
    this.skip = 0;
    this.displayMessage = response.message;
    this.toggleSearchGrid = true;
    this.pageSize = this.displayMessage.includes('between') ? this.searchResults.length : 10;
    //this.pageSize = this.searchResults.length;

    this.searchStateData.lastResults = this.searchResults.slice();
  }

  onSearch(): void {
    if (this.enableSimple) {
      const wrapper = {
        generalSearch: {
          applicantId: this.notaryId.trim()
            ? parseInt(this.notaryId.trim(), 10)
            : null,
          firstName: this.firstName.trim(),
          lastName: this.lastName.trim(),
          cityTown: this.cityTown.trim(),
          approvalDate: this.fromApprovalDate
            ? this.formatDate(this.fromApprovalDate)
            : null,
          dateOfBirth: this.dateOfBirth
            ? this.formatDate(this.dateOfBirth)
            : null,
          remoteNotaryOnly: this.remoteNotariesOnly
        }
      };

      this.search.searchNotaries(wrapper).subscribe({
        next: (response) => {
          this.displayMessage = response.message;
          this.searchResults = response.notarySearchResultsInternalDto;
          this.toggleSearchGrid = true;
          this.skip = 0;
          //to persist the data
          this.searchStateData.lastCriteria = wrapper;
          this.searchStateData.lastResults = response.notarySearchResultsInternalDto?.slice();
        },
        error: (error) => console.error('Error:', error)
      });
      return;
    }
    if (this.enableDateRange) {
      console.log(this.enableDateRange);
      if (this.fromApprovalDate && !this.toApprovalDate) {
        this.toApprovalDate = new Date();
      }

      const wrapper = {
        dateRangeSearch: {
          approvalDateFrom: this.fromApprovalDate
            ? this.formatDate(this.fromApprovalDate)
            : null,
          approvalDateTo: this.toApprovalDate
            ? this.formatDate(this.toApprovalDate)
            : null,
          searchType: Number(this.recordType)
        }
      }

      this.search.searchNotariesWithDateRange(wrapper)
        .subscribe(response => {
          console.log("Here");
          const raw = response.notarySearchResultsInternalDto;
          this.searchResults = raw
            .map((item: any) => ({                   
              ...item,
              approvalDate: item.approvalDate
                ? new Date(item.approvalDate)
                : null,
              createdDate: item.createdDate
                ? new Date(item.createdDate)
                : null
            }));
            
          this.displayMessage = response.message;
          this.pageSize = response.length;
          this.toggleSearchGrid = true;
          //this.skip = 0;
          //this.pageSize = this.searchResults.length;
          //this.toggleSearchGrid = true;
          //to persist data
          this.searchStateData.lastCriteria = wrapper;
          this.searchStateData.lastResults = response.notarySearchResultsInternalDto?.slice();
        },
          err => {
            console.error(err);
          }
        );
      return;
    }
  }

  public onNotaryIdInput(event: Event): void {
    const val = (event.target as HTMLInputElement).value;
    if (!this.hasClearedSimpleFields && val.length === 1) {
      this.firstName = '';
      this.lastName = '';
      this.cityTown = '';
      this.remoteNotariesOnly = false;
      this.fromApprovalDate = null;
      this.toApprovalDate = null;
      this.hasClearedSimpleFields = true;
    }
  }

  onClear(): void {
    // Reset all form fields.
    this.notaryId = '';
    this.cityTown = '';
    this.remoteNotariesOnly = false;
    this.firstName = '';
    this.lastName = '';
    this.fromApprovalDate = null;
    this.dateOfBirth = null;
    this.toApprovalDate = null;
    this.notaryIdError = '';
    this.hasClearedSimpleFields = false; 
  }

  onReset(): void {
    this.toggleSearchGrid = false;
    this.onClear();
  }

  onAddNewRecord(): void {
    this.router.navigate(['add-new-record']);
  }

  // Determines whether all input fields are empty.
  isFormEmpty(): boolean {
    return (
      !this.notaryId.trim() &&
      !this.firstName.trim() &&
      !this.lastName.trim() &&
      !this.cityTown.trim() &&
      !this.dateOfBirth &&
      !this.fromApprovalDate
    );
  }

  // Validates Notary ID on blur. Clears the field and sets an error if not exactly 3 digits.
  validateNotaryId(): void {
    if (this.notaryId && this.notaryId.length < 6) {
      this.notaryIdError = "Notary ID search parameter must be a minimum of 6 digits";
      this.notaryId = '';
    } else {
      this.notaryIdError = "";
    }
  }

  // Restricts key presses to numbers only.
  allowOnlyNumbers(event: KeyboardEvent): void {
    const charCode = event.which ? event.which : event.keyCode;
    if (charCode > 31 && (charCode < 48 || charCode > 57)) {
      event.preventDefault();
    }
  }

  // Helper function to format a Date object as "YYYY-MM-DD"
  private formatDate(date: Date): string {
    const year = date.getFullYear();
    const month = (date.getMonth() + 1).toString().padStart(2, '0');
    const day = date.getDate().toString().padStart(2, '0');
    return `${year}-${month}-${day}`;
  }

  private parseLocalDate(dateStr: string): Date {
    const [year, month, day] = dateStr
      .split('-')
      .map(part => parseInt(part, 10));
    return new Date(year, month - 1, day);
  }

  public pageChange(event: PageChangeEvent): void {
    this.skip = event.skip;
    this.pageSize = event.take;
  }

  public handleNotaryExpand(): void {
    this.isNotaryExpanded = true;
    this.isDateRangeExpanded = false;
    this.enableDateRange = false;
    this.enableSimple = true;
    this.onReset();
  }

  public handleNotaryCollapse(): void {
    this.isNotaryExpanded = false;
    this.isDateRangeExpanded = true;
    this.enableDateRange = true;
    this.enableSimple = false;
    this.onReset();
  }

  public handleDateRangeExpand(): void {
    this.isDateRangeExpanded = true;
    this.isNotaryExpanded = false;
    this.enableDateRange = true;
    this.enableSimple = false;
    this.recordType = "3";
    this.onReset();
  }

  public handleDateRangeCollapse(): void {
    this.isDateRangeExpanded = false;
    this.isNotaryExpanded = true;
    this.enableDateRange = false;
    this.enableSimple = true;
    this.onReset();
  }
}
