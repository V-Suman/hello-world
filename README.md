Bug: When selecting the business address in the add-new-record page and then adding data in there and submitting.. 
there is an error that is popping up. 
Error object: picture attached 
My thoughts: I can see thrugh the error that city field is not being passed, the streetname is not being passed, the zip code
is not being passed? It's confusing. I add those data on the frontend. I am also pasting the request object. 
request object:{
    "personalInfoDto": {
        "salutationTypeId": 5,
        "firstName": "asdsd",
        "middleName": null,
        "lastName": "Savini",
        "suffix": null,
        "dateOfBirth": "1994-07-14",
        "contactsDto": [
            {
                "contactTypeId": 1,
                "contactValue": "1231231231",
                "isPrimary": true
            },
            {
                "contactTypeId": 2,
                "contactValue": "asdsad@asd.asd",
                "isPrimary": false
            }
        ],
        "addressDto": [
            {
                "addressTypeId": 1,
                "isPrefered": false,
                "isPoBox": false,
                "streetNumber": null,
                "streetName": null,
                "aptNumber": null,
                "addressLine2": null,
                "zipCode": "",
                "zipPlus": null,
                "city": "",
                "stateId": 0
            },
            {
                "addressTypeId": 2,
                "isPrefered": true,
                "isPoBox": false,
                "streetNumber": "212",
                "streetName": "dasasa",
                "aptNumber": null,
                "addressLine2": null,
                "zipCode": "12312",
                "zipPlus": null,
                "city": "asdadasd",
                "stateId": 5
            }
        ]
    }
}
when the request object contains data and is correct.. why is it not getting posted to the backend? What is the issue here? 
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
              console.log(request);
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
                  //alert('There was an issue trying to create the new record.');
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
new-notary-record.service.ts file: 
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { AddNewRecordRequest } from '../../models/add-new-record-model/add-new-record-request.model';
import { environment } from '../../../environments/environment';

export interface AddNotaryResponse {
  code: string;
  message: string;
}

@Injectable({
  providedIn: 'root'
})
export class NewNotaryRecordService {
  private readonly endpoint = `${environment.apiurl}/api/InternalUser/AddNewNotary`;

  constructor(private http: HttpClient) { }

  addNewNotaryRecord(request: AddNewRecordRequest): Observable<AddNotaryResponse> {
    return this.http.post<AddNotaryResponse>(this.endpoint, request);
  }
}
