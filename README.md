Goal: To build and console.log the final update personal information object when the user checks the update personal information checkbox 
and subsequently presses the submit button. 

Rules: When the user clicks the udpate personal information chekbox and then proceeds to make changes in the input fields OR may not make 
any changes in the input fields and hits the submit button.. the details present in the input fields need to be packaged into an object and 
should be console.logged.
The final object into which the form data needs to be packed is an interface available in update-profile-info.model.ts file called 
UpdatePersonalProfileInformation.. so for each input fields in the update personal information section 
Ask clarifying questions before you start implementing.
export interface UpdatePersonalProfileInformation
{
  applicantId: number;
  accountId: number;
  salutationTypeId: number;
  firstName: string;
  middleName: string;
  lastName: string;
  suffix: string;
  dateOfBirth: string;
  dateOfDeath: string;
  resignationDate: string;
  isOfficalNameChange: boolean;
  isNameCorrection: boolean;
  contactsDto: [
    {
      contactId: number;
      applicantId: number;
      contactTypeId: number;
      contactValue: string;
      isPrimary: boolean;
    }
  ]
}
1. Salutation: Whatever the user clicks there.. the data will be saved as text.. so you will have to use the text selected by the user then 
query it against the salutationOptions and pick the salutationId relating to that and pass the it to salutationId. Example: If the user 
selected "Dr." as salutation.. then pick the string Dr. then search the salutationOptions to get the salutationId for Dr. and use that salutationId
as salutationId for the UpdatePersonalProfileInformation. This is salutationRef just in case 
export interface SalutationRef {
  salutationTypeId: number;
  code: string;
  value: string;
  description: string;
}
2. First Name, Middle Name, Last Name and Suffix are pretty straightforward. The values of those.. need to be saved into the UpdatePersonalProfileInformation
as firstName, lastName, middleName and suffix.
3. Email Address field. Now, this is a bit tricky. When an email exists.. I want you to pick the email from the form saved values.. and then 
build the contactsDto's member like this. Example: If the email already exists when update-notary-profile-information page loads and is 
testt@tedst.com with applicantId as 1690 then the email contact would look like this     {
      "contactId": 0,
      "applicantId": 1690,
      "contactTypeId": 2,
      "contactValue": "testt@tedst.com",
      "isPrimary": false
    }
Remember the isPrimary flag is always false for an email contact. And the contactId that you see there is also available for you upstream. 
If it is not available (It only won't be ever available if the email is not available).. you will have to set 0 for the contactId. 
Example: if the email does not exist when update-notary-profile-information page loads.. while building the contactsDto.. set the contactId
to 0 and enter whatever the user mentioned in the email address field. 
4. Primary Phone.. The data will be split into 3 input fields phone 1, phone 2 and phone 3.. for a phone number 123-456-7891 I want the contactsDto to 
display it like this     {
      "contactId": 3354 (taken from notary-profile-information),
      "applicantId": 1690, (can be picked from as a result of displayData?.personalInfoDetails?.notaryIdentifier) 
      "contactTypeId": 1 (is always 1 for phone numbers),
      "contactValue": "6175523457" (is the phone number),
      "isPrimary": true (is true for primary phone number)
    }
5. Secondary Phone.. same as primary but secondary phone the isPrimary will be set to false at all costs. 
6. Date of Birth, Date of Death and Date of Resignation should be saved in a format (I think this is SQL formatting) for example if the user 
selected 08/12/1980 the dateOfBirth or dateOfDeath or resignationDate should display "Tue Aug 12 1980 00:00:00 GMT-0400 (Eastern Daylight Time)"
as the value. Every date they select.. turn it into the above date time representation and then push that into the respective UpdatePersonalProfileInformation
properties. 
7. Type of change.. when the user selects either of the official or correction radiobuttons we will have to check the respective boolean to true
or false and the flags are isOfficalNameChange and isNameCorrection 

So in essence when the 
Salutation is Ms.
First Name is Lynda
Middle Name is Empty
Last Name is Maisonnevee
Suffix is Empty
Date Of Birth is 06/18/1988
Email Address arrived Empty but entered test@test.com
Primary Phone is 123-456-7890
Secondary Phone is Empty 
Type of change is Official 
the end UpdatePersonalProfileInformation will look somewhat like: 
{
  applicantId: 854053;
  accountId: 73915 (will be available in the displayData object);
  salutationTypeId: 5;
  firstName: Lynda;
  middleName: null;
  lastName: Maisonnevee;
  suffix: null;
  dateOfBirth: Sat June 18 1988 00:00:00 GMT -0400 (Eastern Datlight Time);
  dateOfDeath: null;
  resignationDate: null;
  isOfficalNameChange: true;
  isNameCorrection: false;
  contactsDto: [
    {
      contactId: 199935 (will be available in the displayData object);
      applicantId: 854053;
      contactTypeId: 1;
      contactValue: "1234567890";
      isPrimary: true;
    },
	{
      contactId: 3354;
      applicantId: 854053;
      contactTypeId: 1;
      contactValue: 1232567890;
      isPrimary: false;
    },
	{
      contactId: 0;
      applicantId: 854053;
      contactTypeId: 2;
      contactValue: test@test.com;
      isPrimary: false;
    }
  ]
}
ts file: 
import { Component } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import mockData from '../../../../mockdata/mock2.json';
import mockProfileData from '../../../../mockdata/mock-profile-data.json';
import { AbstractControl, FormBuilder, FormGroup, ValidationErrors, ValidatorFn, Validators } from '@angular/forms';
import { RefGetterService } from '../../../services/helper-services/ref-get-service/ref-getter.service';
import { CityCountyRef, SalutationRef, StateRef } from '../../../models/ref-items-model/ref-items.model';
import { take } from 'rxjs';
import { BuildEditObject } from '../../../models/update-profile-info-model/update-profile-info.model';

interface NotaryRecord {
  notaryIdentifier: number;
  lastName: string;
  firstName: string;
  middleName: string;
  cityTown: string;
  county: string;
  approvalDate: string;
  createdDate: string;
  isRemoteNotary: string;
  salutationValue: string;
}
interface NotaryProfileRecord {
  notaryDetailInformation: NotaryDetailInformation;
  notaryAccountDetails: NotaryAccountDetails;
}
interface NotaryAccountDetails {
  notaryStatus: string;
  comissionDate: string;
  expirationDate: string;
  hasResigned: string;
}
interface NotaryDetailInformation {
  notaryIdentifier: number;
  dateOfBirth: string;
  phone1: string;
  phone2: string;
  dateOfDeath: string;
  residentialAddress: string;
  businessAddress: string;
  emailAddress: string;
}

@Component({
  selector: 'app-update-notary-profile-info',
  templateUrl: './update-notary-profile-info.component.html',
  styleUrls: ['./update-notary-profile-info.component.css', '../add-new-record/add-new-record.component.css']
})
export class UpdateNotaryProfileInfoComponent {
  public personalForm!: FormGroup;
  public addressForm!: FormGroup;

  public record?: NotaryRecord;
  public detailInfo?: NotaryDetailInformation;
  public accountInfo?: NotaryAccountDetails;

  // start empty, we’ll overwrite once the call returns
  public salutationOptions: string[] = [];
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
    private route: ActivatedRoute
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

    this.record = (mockData as NotaryRecord[])
      .find(r => r.notaryIdentifier === notaryId);

    const profile = (mockProfileData as NotaryProfileRecord[])
      .find(p => p.notaryDetailInformation.notaryIdentifier === notaryId);

    if (profile) {
      this.detailInfo = profile.notaryDetailInformation;
      this.accountInfo = profile.notaryAccountDetails;
    }

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
      changeType: [null]
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
        dateOfResignation: null,          // always blank

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
    this.refGetter.getSalutations().subscribe(
      (all: SalutationRef[]) => {
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

  public onSubmit(): void {
    this.submittedAddress = true;
    if (!this.updatePersonalChecked && !this.updateAddressChecked) {
      return;
    }
    if (this.updatePersonalChecked) {
      if (this.personalForm.valid) {
        console.log('Personal Information:', this.personalForm.value);
      } else {
        console.warn('Personal form is invalid:', this.personalForm.errors);
      }
    }
    if (this.updateAddressChecked) {
      if (this.addressForm.valid) {
        console.log('Address Information:', this.addressForm.value);
      } else {
        console.warn('Address form is invalid:', this.addressForm.errors);
      }
    }
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
