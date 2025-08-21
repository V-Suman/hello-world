The goal is to build a buildUpdateComplaintObject and console.log it whenever I call it. Simple. 
Important to note here is that the fields needed for building the buildUpdateComplaintObject are already available via different calls 
throughout the page. I just want to pick from them and use them. 

Simple question. The ComplaintHistory interface has complaintId etc etc in the model.ts file. Now tell me why tf am I getting an error on 
line 121,122,123,124..127 which says TS2339: Property 'complaintId' does not exist on type 'ComplaintHistory[]' 

Go through the code and ask clarifying questions before you start implementing. 

Here is the component.ts file: 
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
      return;
    }

    this.notaryId = Number(idParam);
    if (isNaN(this.notaryId)) {
      console.error(`Route parameter "id" is not a valid number: ${idParam}`);
      return;
    }

    this.notaryDetails.getNotaryProfile(Number(idParam)).subscribe({
      next: response => {
        this.profileDetail = this.mapToDetailInfo(response);
        this.accountDetail = this.mapToAccountDetailInfo(response);
        this.applicationDetail = this.mapToApplicationDetailInfo(response);
        this.buildEditObject = this.mapToEditObject(response);
        console.log(this.buildEditObject);
      },
      error: err => {
        console.error('Error fetching notary profile:', err);
      }
    });

  }

  private mapToEditComplaintObject(resp: any): UpdateComplaint {
    const pid = resp.personalInfoDetails;
    const cid = this.complaintHistory;
    return {
      firstName: pid.firstName,
      middleName: pid.middleName,
      lastName: pid.lastName,
      applicantId: pid.applicantId,
      accountId: resp.accountDetails?.accountId,
      complaintId: cid?.complaintId,
      dateOfComplaint: cid?.dateOfComplaint,
      isRoncomplaint:cid?.isRoncomplaint,
      complaintDetails:cid?.complaintDetails,
      isResolved:cid?.isResolved,
      resolutionNotes:cid?.resolutionNotes,
      resolutionDate: cid?.resolutionDate
    }
  }

  private mapToEditObject(resp: any): BuildEditObject {
    const pid = resp.personalInfoDetails;
    const email = resp.emailAddress;
    const accountIdentifier = resp.accountDetails?.accountId;

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
    const name = [pid.firstName, pid.middleName, pid.lastName]
      .filter(n => !!n)
      .join(' ');

    // 2 & 10) Dates
    const dateOfBirth = pid.dateOfBirth.split('T')[0];
    const dateOfDeath = pid.dateOfDeath
      ? pid.dateOfDeath.split('T')[0]
      : '---';

    // 3) Email
    const emailAddress = resp.emailAddress!=null ? resp.emailAddress : '---';

    // 4 & 5) Phones
    const primaryPhone = phones.find(p => p.isPrimary)!.contactValue;
    const secondary = phones.find(p => !p.isPrimary);
    const phone1 = primaryPhone;
    const phone2 = secondary ? secondary.contactValue : null;

    // Preferred address for county & district
    const preferred = addrs.find(a => a.isPrefered)!;
    const countyName = preferred.county;
    const districtName = preferred.district;

    // Helper to format an address object
    const formatAddress = (a: any) => {
      const parts: string[] = [];
      if (a.streetNumber) parts.push(a.streetNumber);
      if (a.streetName) parts.push(a.streetName);
      if (a.aptNumber) parts.push(a.aptNumber);
      if (a.addressLine2) parts.push(a.addressLine2);
      if (a.city) parts.push(a.city);
      if (a.state && a.zipCode) {
        const zp = a.zipPlus ? `-${a.zipPlus}` : '';
        parts.push(`${a.state}-${a.zipCode}${zp}`);
      }
      return parts.join(',');
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
            updatedOn: '---',
            firstName: '---',
            middleName: '',
            lastName: '',
            dateOfBirth: '---',
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
          updatedOn: '---',
          firstName: '---',
          middleName: '',
          lastName: '',
          dateOfBirth: '---',
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
    return '---';
  }

  public onAddressExpand(): void {
    if (this.addressHistoryLoaded) { return; }

    this.addressHistoryService.getAddressHistory(this.notaryId).subscribe({
      next: (data: AddressHistory[]) => {
        if (!data || data.length === 0) {
          // one row of all '---'
          this.addressHistory = [{
            updatedOn: '---',
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
          updatedOn: '---',
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
              dateOfComplaint: '---',
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
            dateOfComplaint: '---',
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
      return '---';
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
      applicationNextStep: app.applicationNextStep=='' ? '---' : app.applicationNextStep,
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
            approvalDate: '---',
            expirationDate: '---',
            resignationDate: '---',
            paymentDate: '---',
            qualifiedDate: '---',
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
          approvalDate: '---',
          expirationDate: '---',
          resignationDate: '---',
          paymentDate: '---',
          qualifiedDate: '---',
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
            validationStartDate: '---',
            validationEndDate: '---',
            createdOn: '---',
            createdByUser: '---'
          }];
        }
      });
  }


  public returnToSearch(): void {
    this.router.navigate(['/notary-records']);
  }
}
here is the code in ref-items.model.ts file: 
export interface ComplaintHistory {
  complaintId: number;
  dateOfComplaint: string;
  complaintDetails: string;
  isRoncomplaint: string;
  isResolved: boolean;
  resolutionNotes: string;
  resolutionDate: string;
}
