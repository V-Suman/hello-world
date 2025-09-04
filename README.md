Goal: To handle scenario where the notary profile does not have accountDetails. 
Details: A general notary profile response looks like this.. {
  "emailAddress": null,
  "personalInfoDetails": {
    "salutationTypeId": 5,
    "salutationType": "Ms.",
    "firstName": "Diane",
    "middleName": "M.",
    "lastName": "Smith",
    "suffix": null,
    "dateOfBirth": "1959-02-18T00:00:00",
    "applicantId": 1429,
    "notaryIdentifier": 1429,
    "dateOfDeath": null,
    "addressDetails": [
      {
        "addressId": 337630,
        "applicantId": 1429,
        "addressTypeId": 1,
        "addressType": "Residential",
        "isPrefered": true,
        "isPoBox": false,
        "streetNumber": "23",
        "streetName": "Karen Ave",
        "aptNumber": "",
        "addressLine2": null,
        "zipCode": "02053",
        "zipPlus": null,
        "city": "Medway",
        "county": "Norfolk",
        "district": "Second District",
        "stateId": 20,
        "state": "MA"
      }
    ],
    "contactDetails": [
      {
        "contactTypeId": 1,
        "contactType": "Phone",
        "contactId": 3046,
        "applicantId": 1429,
        "contactValue": "5085331670",
        "isPrimary": true
      },
      {
        "contactTypeId": 1,
        "contactType": "Phone",
        "contactId": 200410,
        "applicantId": 1429,
        "contactValue": "5082944074",
        "isPrimary": false
      }
    ]
  },
  "accountDetails": {
    "accountId": 8752,
    "applicantId": 1429,
    "accountStatusId": 2,
    "accountStatus": "Active",
    "approvalDate": "2024-03-13T00:00:00",
    "expirationDate": "2031-03-13T00:00:00",
    "hasResigned": false,
    "resignationDate": null,
    "isRemoteNotary": false
  },
  "applicationDetails": {
    "applicationId": 2043,
    "applicantId": 1429,
    "applicationType": null,
    "applicationTypeId": 2,
    "applicationDate": "2024-05-24T09:28:16.03",
    "applicationStatusDate": null,
    "applicationStatus": "Completed",
    "applicationStatusId": 15,
    "nextApplicationStatusId": null,
    "applicationNextStep": "",
    "exceptionApplicationStatusId": null,
    "exceptionApplicationStatus": "",
    "approvalDate": "2024-03-13",
    "dueDate": "2024-06-13",
    "applicationStatusToolTip": "",
    "isRONApproved": false,
    "roN_Partners": null
  },
  "hasActiveComplaints": false,
  "displayControls": [
    {
      "controlID": 6,
      "displayText": "Apply to become a Remote Online Notary",
      "visible": true,
      "enabled": true,
      "displaySequence": 4
    },
    {
      "controlID": 4,
      "displayText": "Update Login Information",
      "visible": true,
      "enabled": true,
      "displaySequence": 6
    },
    {
      "controlID": 5,
      "displayText": "Update Profile Information",
      "visible": true,
      "enabled": true,
      "displaySequence": 7
    }
  ]
}
Now notice.. that here we have a sub-object inside called the accountDetails object. In some cases this acountDetails object will turn up 
empty. And that is ok. It can turn up empty. An example of a notary with accountDetails object empty is as follows:
{
  "emailAddress": "asdas@asdsa.asd",
  "personalInfoDetails": {
    "salutationTypeId": 3,
    "salutationType": "Mr.",
    "firstName": "Jeremy",
    "middleName": null,
    "lastName": "Driver",
    "suffix": null,
    "dateOfBirth": "1944-05-14T00:00:00",
    "applicantId": 1000017,
    "notaryIdentifier": 1000017,
    "dateOfDeath": null,
    "addressDetails": [
      {
        "addressId": 390938,
        "applicantId": 1000017,
        "addressTypeId": 1,
        "addressType": "Residential",
        "isPrefered": true,
        "isPoBox": false,
        "streetNumber": "212",
        "streetName": "dasasa",
        "aptNumber": "12",
        "addressLine2": null,
        "zipCode": "12312",
        "zipPlus": null,
        "city": "asasdsad",
        "county": null,
        "district": null,
        "stateId": 6,
        "state": "CO"
      }
    ],
    "contactDetails": [
      {
        "contactTypeId": 1,
        "contactType": "Phone",
        "contactId": 358492,
        "applicantId": 1000017,
        "contactValue": "1231313131",
        "isPrimary": true
      },
      {
        "contactTypeId": 2,
        "contactType": "Email",
        "contactId": 358493,
        "applicantId": 1000017,
        "contactValue": "asdas@asdsa.asd",
        "isPrimary": false
      }
    ]
  },
  "accountDetails": null,
  "applicationDetails": {
    "applicationId": 800020,
    "applicantId": 1000017,
    "applicationType": null,
    "applicationTypeId": 1,
    "applicationDate": "2025-09-04T19:14:04.29",
    "applicationStatusDate": "2025-09-04T19:14:04.29",
    "applicationStatus": "Pending",
    "applicationStatusId": 2,
    "nextApplicationStatusId": 3,
    "applicationNextStep": "Approved",
    "exceptionApplicationStatusId": 4,
    "exceptionApplicationStatus": "Rejected",
    "approvalDate": null,
    "dueDate": null,
    "applicationStatusToolTip": "Your application has been submitted and is in 'Pending' status. Once your application is reviewed, you will be notified by email and US Mail to the corresponding address on your record. If you are approved, instructions and next steps will be provided. Please note, approvals generally take between 1 to 3 weeks.",
    "isRONApproved": false,
    "roN_Partners": null
  },
  "hasActiveComplaints": false,
  "displayControls": [
    {
      "controlID": 4,
      "displayText": "Update Login Information",
      "visible": true,
      "enabled": true,
      "displaySequence": 6
    }
  ]
}
Now, the scenario is.. when the accountDetails object is empty.. we will not have the accountId property value.. and that property 
value is very crucial in downstream systems like update-notary-details. So..
I need you to use the value '0' as accountId to downstream systems like update-notary-details and update-personal-information whenever the 
accountDetails sub-object is empty. Such that.. when I am in update-notary-details for a user with no accountDetails I can still add notes 
to them via mentioning that their accountId is 0. 

Ask me all clarifying questions before you get started. 
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
                accountId: this.accountId ?? 0,   // send 0 when missing
                notes: (this.notesCtrl.value ?? '').toString()
            };
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

        if (this.validCertificateEnabledCtrl.value) {
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

        if (this.addComplaintEnabledCtrl.value) {
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
