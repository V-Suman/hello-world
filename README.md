Goal: Search the getNotaryProfile response object and enable the respective tooltip based on a property. 
Details: Currently the getNotaryProfile response object looks like this 
{
    "emailAddress": null,
    "personalInfoDetails": {
        "salutationTypeId": 3,
        "salutationType": "Mr.",
        "firstName": "Stephen",
        "middleName": "A.",
        "lastName": "Mangano",
        "suffix": null,
        "dateOfBirth": "1960-09-20T00:00:00",
        "applicantId": 1104,
        "notaryIdentifier": 1104,
        "dateOfDeath": null,
        "addressDetails": [
            {
                "addressId": 273077,
                "applicantId": 1104,
                "addressTypeId": 1,
                "addressType": "Residential",
                "isPrefered": true,
                "isPoBox": false,
                "streetNumber": "29",
                "streetName": "Tower Hill Rd",
                "aptNumber": "",
                "addressLine2": null,
                "zipCode": "01864",
                "zipPlus": null,
                "city": "North Reading",
                "county": "Middlesex",
                "district": "Fifth District",
                "stateId": 20,
                "state": "MA"
            }
        ],
        "contactDetails": [
            {
                "contactTypeId": 1,
                "contactType": "Phone",
                "contactId": 4102,
                "applicantId": 1104,
                "contactValue": "5089822868",
                "isPrimary": true
            },
            {
                "contactTypeId": 1,
                "contactType": "Phone",
                "contactId": 201184,
                "applicantId": 1104,
                "contactValue": "9789822868",
                "isPrimary": false
            }
        ]
    },
    "accountDetails": {
        "accountId": 12828,
        "applicantId": 1104,
        "accountStatusId": 2,
        "accountStatus": "Active",
        "approvalDate": "2022-10-12T00:00:00",
        "expirationDate": "2029-10-12T00:00:00",
        "hasResigned": false,
        "resignationDate": null,
        "isRemoteNotary": false
    },
    "applicationDetails": {
        "applicationId": 1555,
        "applicantId": 1104,
        "applicationType": null,
        "applicationTypeId": 2,
        "applicationDate": "2022-11-23T11:11:06.47",
        "applicationStatusDate": null,
        "applicationStatus": "Completed",
        "applicationStatusId": 15,
        "nextApplicationStatusId": null,
        "applicationNextStep": "",
        "exceptionApplicationStatusId": null,
        "exceptionApplicationStatus": "",
        "approvalDate": "2022-10-12",
        "dueDate": "2023-01-12",
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
inside the response.. there is the personalInfoDetails which has the addressDetails object 
which can contain at most 2 addresses business and residential. Oout of which one of them will 
have the flag isPrefered as true and the other one will have false. 

We need to see which address has the value as true and enable that tooltip boolean. 

For the above example.. there is only one address and that address has the property isPrefered
as true and that address is residential. So.. we set the showResidentialToolTip as true. 

{
    "emailAddress": null,
    "personalInfoDetails": {
        "salutationTypeId": 3,
        "salutationType": "Mr.",
        "firstName": "Stephen",
        "middleName": "A.",
        "lastName": "Mangano",
        "suffix": null,
        "dateOfBirth": "1960-09-20T00:00:00",
        "applicantId": 1104,
        "notaryIdentifier": 1104,
        "dateOfDeath": null,
        "addressDetails": [
            {
                "addressId": 273077,
                "applicantId": 1104,
                "addressTypeId": 1,
                "addressType": "Residential",
                "isPrefered": false,
                "isPoBox": false,
                "streetNumber": "29",
                "streetName": "Tower Hill Rd",
                "aptNumber": "",
                "addressLine2": null,
                "zipCode": "01864",
                "zipPlus": null,
                "city": "North Reading",
                "county": "Middlesex",
                "district": "Fifth District",
                "stateId": 20,
                "state": "MA"
            },
			{
                "addressId": 273047,
                "applicantId": 1104,
                "addressTypeId": 2,
                "addressType": "Business",
                "isPrefered": true,
                "isPoBox": false,
                "streetNumber": "10",
                "streetName": "Bodda Hill Rd",
                "aptNumber": "",
                "addressLine2": null,
                "zipCode": "01864",
                "zipPlus": null,
                "city": "North Reading",
                "county": "Middlesex",
                "district": "Fifth District",
                "stateId": 20,
                "state": "MA"
            },
        ],
        "contactDetails": [
            {
                "contactTypeId": 1,
                "contactType": "Phone",
                "contactId": 4102,
                "applicantId": 1104,
                "contactValue": "5089822868",
                "isPrimary": true
            },
            {
                "contactTypeId": 1,
                "contactType": "Phone",
                "contactId": 201184,
                "applicantId": 1104,
                "contactValue": "9789822868",
                "isPrimary": false
            }
        ]
    },
    "accountDetails": {
        "accountId": 12828,
        "applicantId": 1104,
        "accountStatusId": 2,
        "accountStatus": "Active",
        "approvalDate": "2022-10-12T00:00:00",
        "expirationDate": "2029-10-12T00:00:00",
        "hasResigned": false,
        "resignationDate": null,
        "isRemoteNotary": false
    },
    "applicationDetails": {
        "applicationId": 1555,
        "applicantId": 1104,
        "applicationType": null,
        "applicationTypeId": 2,
        "applicationDate": "2022-11-23T11:11:06.47",
        "applicationStatusDate": null,
        "applicationStatus": "Completed",
        "applicationStatusId": 15,
        "nextApplicationStatusId": null,
        "applicationNextStep": "",
        "exceptionApplicationStatusId": null,
        "exceptionApplicationStatus": "",
        "approvalDate": "2022-10-12",
        "dueDate": "2023-01-12",
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
In the above example there are 2 addresses and one of the address has isPrefered as true and 
we set that address's tooltip as true which is business.

ASk me any clarifying questions before you start. 
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
    primaryAddress: string = "This is the user's preferred address";
    showResidentialToolTip: boolean = false;
    showBusinessToolTip: boolean = false;
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
                console.log(this.profileDetail);
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
                    console.log("Name Data:", data);
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
