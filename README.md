I am utterly confused, here are the ts and my service files. Give me the entire function block 
to slot in if you have any changes. 

component.ts file: 
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import mockData from '../../../../mockdata/mock2.json';
import mockProfileData from '../../../../mockdata/mock-profile-data.json';
import { AbstractControl, FormBuilder, FormGroup, ValidationErrors, ValidatorFn, Validators } from '@angular/forms';
import { RefGetterService } from '../../../services/helper-services/ref-get-service/ref-getter.service';
import { CityCountyRef, SalutationRef, StateRef } from '../../../models/ref-items-model/ref-items.model';
import { take } from 'rxjs';
import { BuildEditObject, UpdateAddressProfileInformation, UpdatePersonalProfileInformation } from '../../../models/update-profile-info-model/update-profile-info.model';
import { UpdateNotaryProfileInformationService } from '../../../services/update-notary-profile-info/update-notary-profile-information.service';

@Component({
    selector: 'app-update-notary-profile-info',
    templateUrl: './update-notary-profile-info.component.html',
    styleUrls: ['./update-notary-profile-info.component.css', '../add-new-record/add-new-record.component.css']
})
export class UpdateNotaryProfileInfoComponent implements OnInit {
    public personalForm!: FormGroup;
    public addressForm!: FormGroup;
    showValidationErrors: boolean = false;
    // start empty, we’ll overwrite once the call returns
    public salutationOptions: string[] = [];
    public salutationRefs: SalutationRef[] = [];
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
    private poBoxMemory: { number: string } = { number: '' };

    constructor(
        private fb: FormBuilder,
        private refGetter: RefGetterService,
        private router: Router,
        private route: ActivatedRoute,
        private updateService: UpdateNotaryProfileInformationService
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
        // your existing route + mock-data lookup logic 
        const idParam = this.route.snapshot.paramMap.get('id');
        if (!idParam) { console.error('No "id" in route!'); return; }
        const notaryId = Number(idParam);
        if (isNaN(notaryId)) { console.error(`Bad id: ${idParam}`); return; }

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
            changeType: [null,[Validators.required]]
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
                dateOfResignation: p.dateOfResignation ? new Date(p.dateOfResignation) : null,

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
        }

        this.originalPersonalValues = this.personalForm.value;
        this.personalForm.disable();

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

        this.refGetter.getSalutations().subscribe(
            (all: SalutationRef[]) => {
                this.salutationRefs = all;                    //  save full array
                this.salutationOptions = all.map(x => x.value);
            },
            err => console.error('Failed to load salutations', err)
        );

        this.getStatesForDropDown();
    }

    private parsePoBoxNumber(streetName: string | null | undefined): string {
        if (!streetName) return '';
        const m = /^PO\s*Box\s*(.+)$/i.exec(streetName.trim());
        return m ? m[1].trim() : '';
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
                isPoBox: [false],
                street1: [''],
                street2: [''],
                street3: [''],
                street4: [''],
                state: [''],
                zipCode: ['', [Validators.required, Validators.pattern(/^\d{5}$/)]],
                zipPlus: ['', Validators.pattern(/^$|^\d{2,4}$/)],
                cityTown: ['']
            }, { validators: this.streetValidator() })
        });


        this.addressForm.get('preferredAddress')!
            .valueChanges
            .subscribe((pref: 'Residence' | 'Business') => this.onPreferredAddressChange(pref));

        // init validators for default “Residence”
        this.updateAddressValidators(this.addressForm.get('preferredAddress')!.value);
    }

    private onPreferredAddressChange(pref: 'Residence' | 'Business'): void {
        const res = this.addressForm.get('residenceAddress') as FormGroup;
        const bus = this.addressForm.get('businessAddress') as FormGroup;

        // snapshot current selections so they don't get lost on revalidation / UI flip
        const keepResState = res.get('state')!.value;
        const keepResCity = res.get('cityTown')!.value;
        const keepBusState = bus.get('state')!.value;
        const keepBusCity = bus.get('cityTown')!.value;

        // reconfigure validators for the new preferred side
        this.updateAddressValidators(pref);

        // silently restore selections so dropdowns don't clear
        res.get('state')!.setValue(keepResState, { emitEvent: false });
        res.get('cityTown')!.setValue(keepResCity, { emitEvent: false });
        bus.get('state')!.setValue(keepBusState, { emitEvent: false });
        bus.get('cityTown')!.setValue(keepBusCity, { emitEvent: false });

        // refresh MA city list without blowing away selected city
        this.updateCityOptions();
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
            //resG.setValidators(this.optionalGroupValidator());
        }

        // re-run
        [resG, busG].forEach(g => g.updateValueAndValidity({ onlySelf: true }));
        ['state', 'zipCode', 'cityTown', 'zipPlus']
            .forEach(n => {
                resG.get(n)?.updateValueAndValidity({ onlySelf: true });
                busG.get(n)?.updateValueAndValidity({ onlySelf: true });
            });
    }

    public onPoBoxToggle(evt: Event): void {
        const checked = (evt.target as HTMLInputElement).checked;

        const res = this.addressForm.get('residenceAddress') as FormGroup;
        const bus = this.addressForm.get('businessAddress') as FormGroup;

        // Preserve both groups' state/city across the UI shape flip
        const keepResState = res.get('state')!.value;
        const keepResCity = res.get('cityTown')!.value;
        const keepBusState = bus.get('state')!.value;
        const keepBusCity = bus.get('cityTown')!.value;

        // If turning OFF PO Box, remember the current PO number
        if (!checked) {
            const current = (bus.get('street3')!.value || '').toString().trim();
            if (current) this.poBoxMemory.number = current;
        }

        bus.get('isPoBox')!.setValue(checked, { emitEvent: false });
        this.setPoBoxMode(checked);

        // Restore states/cities silently so nothing blanks out
        res.get('state')!.setValue(keepResState, { emitEvent: false });
        res.get('cityTown')!.setValue(keepResCity, { emitEvent: false });
        bus.get('state')!.setValue(keepBusState, { emitEvent: false });
        bus.get('cityTown')!.setValue(keepBusCity, { emitEvent: false });

        this.updateCityOptions();
    }


    private setPoBoxMode(on: boolean): void {
        const bus = this.addressForm.get('businessAddress') as FormGroup;

        if (on) {
            // PO-Box UI shape
            bus.get('street1')!.setValue('', { emitEvent: false });        // Street No. not used
            bus.get('street2')!.setValue('PO Box', { emitEvent: false });  // static label
            bus.get('street3')!.setValue(this.poBoxMemory.number || bus.get('street3')!.value || '', { emitEvent: false });

            // Validators for PO mode: require PO number only at group level
            bus.get('street1')!.clearValidators();
            bus.get('street2')!.clearValidators();
            bus.get('street3')!.setValidators([Validators.required]);
            bus.get('street4')!.clearValidators();

            bus.clearValidators();
            bus.setValidators(this.poBoxGroupValidator());
        } else {
            // Non-PO mode so set Business required validators ONLY on the Business group
            bus.get('street1')!.clearValidators();
            bus.get('street2')!.clearValidators();
            bus.get('street3')!.clearValidators();
            bus.get('street4')!.clearValidators();

            bus.clearValidators();
            this.applyBusinessRequiredValidators(); 
        }

        // Finalize
        ['street1', 'street2', 'street3', 'street4'].forEach(n =>
            bus.get(n)!.updateValueAndValidity({ onlySelf: true })
        );
        bus.updateValueAndValidity({ onlySelf: true });
    }


    /** Apply the "Business is the required address" rules without touching Residence. */
    private applyBusinessRequiredValidators(): void {
        const bus = this.addressForm.get('businessAddress') as FormGroup;

        // Group-level requirement: some street (1 or 2)
        bus.setValidators(this.streetValidator());

        // Field-level requirements
        const req = Validators.required;
        const zipReq = [Validators.required, Validators.pattern(/^\d{5}$/)];
        const zipPlusPat = Validators.pattern(/^$|^\d{2,4}$/);

        bus.get('state')!.setValidators(req);
        bus.get('zipCode')!.setValidators(zipReq);
        bus.get('cityTown')!.setValidators(req);
        bus.get('zipPlus')!.setValidators(zipPlusPat);

        // keep street validators free here; group validator enforces street1/street2 presence
        ['state', 'zipCode', 'cityTown', 'zipPlus'].forEach(n =>
            bus.get(n)!.updateValueAndValidity({ onlySelf: true })
        );

        bus.updateValueAndValidity({ onlySelf: true });
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
                this.cacheCityCountyData();
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
        if (!this.displayData) return;
        const addrs = this.displayData.personalInfoDetails.addressDetails;

        const preferred = addrs.find(a => a.isPrefered) || addrs[0];
        const other = addrs.find(a => !a.isPrefered) || null;

        const prefType = preferred.addressType === 'Business' ? 'Business' : 'Residence';
        this.addressForm.get('preferredAddress')!.setValue(prefType);

        const doPatch = (grpName: 'residenceAddress' | 'businessAddress', addr: any) => {
            const grp = this.addressForm.get(grpName) as FormGroup;

            const stateRef = this.stateRefs.find(s => s.stateId === addr.stateId);
            const fullState = stateRef?.value || '';
            grp.get('state')!.setValue(fullState);

            grp.get('cityTown')!.reset();
            grp.get('cityTown')!.setValue(addr.city);

            grp.get('zipCode')!.setValue(addr.zipCode);
            grp.get('zipPlus')!.setValue(addr.zipPlus || '');

            if (grpName === 'businessAddress') {
                grp.get('isPoBox')!.setValue(!!addr.isPoBox, { emitEvent: false });

                if (addr.isPoBox) {
                    this.poBoxMemory.number = this.parsePoBoxNumber(addr.streetName);
                    this.setPoBoxMode(true);

                    grp.patchValue({
                        street2: 'PO Box',
                        street3: this.poBoxMemory.number,
                        street4: addr.addressLine2 || ''
                    }, { emitEvent: false });
                } else {
                    this.setPoBoxMode(false);

                    grp.patchValue({
                        street1: addr.streetNumber || '',
                        street2: addr.streetName || '',
                        street3: addr.aptNumber || '',      // fixed
                        street4: addr.addressLine2 || ''
                    }, { emitEvent: false });
                }
            } else {
                grp.patchValue({
                    street1: addr.streetNumber || '',
                    street2: addr.streetName || '',
                    street3: addr.aptNumber || '',
                    street4: addr.addressLine2 || ''
                }, { emitEvent: false });
            }
        };

        const res = addrs.find(a => a.addressType === 'Residential');
        if (res) doPatch('residenceAddress', res);

        const bus = addrs.find(a => a.addressType === 'Business');
        if (bus) doPatch('businessAddress', bus);

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

    markAllFieldsTouched(formGroup: FormGroup): void {
        Object.keys(formGroup.controls).forEach((key) => {
            const control = formGroup.get(key);
            control?.markAsTouched();
            control?.updateValueAndValidity();
        });
    }

    public onSubmit(): void {
        if (!this.updatePersonalChecked && !this.updateAddressChecked) {
            return;
        }

        this.showValidationErrors = true;

        if (this.updatePersonalChecked) {
            if (this.personalForm.invalid) {
            this.markAllFieldsTouched(this.personalForm); // highlight errors
            return; // stop submission
        }
            if (this.personalForm.valid) {
                const payload = this.buildUpdatePersonalPayload();
                console.log('UpdatePersonalProfileInformation:', payload);
                this.updateService.updatePersonalInformation(payload)
                    .subscribe({
                        next: success => {
                            console.log('UpdatePersonalProfileInformation success:', success);
                            this.router.navigate(['/notary-profile', payload.applicantId]);
                            // maybe show a toast or navigate on success
                        },
                        error: err => {
                            console.error('UpdatePersonalProfileInformation failed:', err);
                            // handle/display errors
                        }
                    });
            } else {
                console.warn('Personal form is invalid:', this.personalForm.errors);
            }
        }
        if (this.updateAddressChecked) {
            this.submittedAddress = true;
            if (this.addressForm.invalid) {

                this.markAllFieldsTouched(this.addressForm); // Highlight errors
                return; // Stop submission
            }
            if (this.addressForm.valid) {
                const addressPayloads = this.buildUpdateAddressPayloads();
                console.log('UpdateAddressProfileInformation:', addressPayloads);
                //addressPayloads.forEach(payload => {
                    
                //});
                const payload = addressPayloads;
                this.updateService.updateAddressInformation(payload).subscribe({
                    next: success => {
                        //console.log(`UpdateAddressProfileInformation success for ${payload[].addressTypeId}:`, success);
                        // Only navigate after the last address update (if needed)
                        const applicantId = this.displayData?.personalInfoDetails.applicantId || this.route.snapshot.paramMap.get('id');
                        this.router.navigate(['/notary-profile', applicantId]);
                    },
                    error: err => {
                        //console.error(`UpdateAddressProfileInformation failed for ${payload.addressTypeId}:`, err);
                        // Handle/display errors (e.g., show a toast)
                    }
                });
            } else {
                console.warn('Address form is invalid:', this.addressForm.errors);
            }
        }
    }
    private buildUpdateAddressPayloads(): UpdateAddressProfileInformation[] {
        const payloads: UpdateAddressProfileInformation[] = [];
        const formValue = this.addressForm.value;
        const preferredAddress = formValue.preferredAddress;
        const buildAddressPayload = (
            addressType: 'Residential' | 'Business',
            formGroup: any,
            isPreferred: boolean,
            originalAddress: any
        ): UpdateAddressProfileInformation => {
            const stateValue = formGroup.state;
            const stateRef = this.stateRefs.find(s => s.value === stateValue);
            const cityValue = formGroup.cityTown;
            const d = this.displayData!.personalInfoDetails;
            const applicantId = d.applicantId;
            const accountId = d.accountId;

            return {
                addressId: originalAddress?.addressId || 0, 
                applicantId: d.applicantId,
                addressTypeId: addressType === 'Residential' ? 1 : 2,
                isPrefered: isPreferred,
                isPoBox: addressType === 'Business' ? formGroup.isPoBox : false,
                streetNumber: formGroup.isPoBox ? '' : formGroup.street1 || '',
                streetName: formGroup.isPoBox ? 'PO Box' : formGroup.street2 || '',
                aptNumber: formGroup.isPoBox ? formGroup.street3 || '' : formGroup.street3 || '',
                addressLine2: formGroup.street4 || '',
                zipCode: formGroup.zipCode || '',
                zipPlus: formGroup.zipPlus || '',
                city: formGroup.cityTown || '',
                county: cityValue || '',
                district: '', // optional here
                stateId: stateRef?.stateId || 0
            };
        };

        const addrs = this.displayData?.personalInfoDetails.addressDetails || [];

        const resAddress = addrs.find(a => a.addressType === 'Residential');
        if (formValue.residenceAddress.state || formValue.residenceAddress.zipCode || formValue.residenceAddress.cityTown) {
            payloads.push(buildAddressPayload(
                'Residential',
                formValue.residenceAddress,
                preferredAddress === 'Residence',
                resAddress
            ));
        }

        const busAddress = addrs.find(a => a.addressType === 'Business');
        if (formValue.businessAddress.state || formValue.businessAddress.zipCode || formValue.businessAddress.cityTown) {
            payloads.push(buildAddressPayload(
                'Business',
                formValue.businessAddress,
                preferredAddress === 'Business',
                busAddress
            ));
        }

        return payloads;
    }


    private buildUpdatePersonalPayload(): UpdatePersonalProfileInformation {
        const f = this.personalForm.value;
        const d = this.displayData!.personalInfoDetails;
        const applicantId = d.applicantId;
        const accountId = d.accountId;

        // 1) lookup salutationTypeId
        const sel = f.salutation as string;
        const salRef = this.salutationRefs.find(r => r.value === sel);
        const salutationTypeId = salRef?.salutationTypeId ?? 0;

        // 2) normalize name fields
        const norm = (v: string | null) => v ? v : null;

        // 3) format dates
        const formatDate = (dt: Date | null): string | null => {
            if (!dt) {
                return null;
            }
            const year = dt.getFullYear();
            const month = String(dt.getMonth() + 1).padStart(2, '0');
            const day = String(dt.getDate()).padStart(2, '0');
            return `${year}-${month}-${day}T00:00:00`;
        };
        const dob = formatDate(f.dateOfBirth);
        const dod = formatDate(f.dateOfDeath);
        const res = formatDate(f.dateOfResignation);

        // 4) change flags
        const isOfficial = f.changeType === 'Official';
        const isCorrection = f.changeType === 'Correction';

        // 5) build contacts array
        const contacts: UpdatePersonalProfileInformation['contactsDto'] = [];
        const makePhone = (blocks: [string, string, string], primary: boolean) => {
            const val = blocks.join('');
            if (!val) return;  // skip if totally blank
            const orig = d.contactDetails.find(c =>
                c.contactType === 'Phone' && c.isPrimary === primary
            );
            contacts.push({
                contactId: orig?.contactId ?? 0,
                applicantId,
                contactTypeId: 1,
                contactValue: val,
                isPrimary: primary
            });
        };
        makePhone([f.primaryPhone1, f.primaryPhone2, f.primaryPhone3], true);
        makePhone([f.secondaryPhone1, f.secondaryPhone2, f.secondaryPhone3], false);

        // 6) email
        const emailVal = (f.email as string || '').trim();
        if (emailVal) {
            const orig = d.contactDetails.find(c => c.contactType === 'Email');
            contacts.push({
                contactId: orig?.contactId ?? 0,
                applicantId,
                contactTypeId: 2,
                contactValue: emailVal,
                isPrimary: false
            });
        }

        // 7) put it all together
        return {
            applicantId,
            accountId,
            salutationTypeId,
            firstName: f.firstName,
            middleName: norm(f.middleName),
            lastName: f.lastName,
            suffix: norm(f.suffix),
            dateOfBirth: dob!,
            dateOfDeath: dod!,
            resignationDate: res!,
            isOfficalNameChange: isOfficial,
            isNameCorrection: isCorrection,
            contactsDto: contacts
        };
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
                this.personalForm.get(name)!.setValue(chunk);
            }
        });

        // focus the first not-full box, or the last one if all are full
        const next = inputs.find(i => i.value.length < i.maxLength) || inputs[inputs.length - 1];
        next.focus();
        next.setSelectionRange(next.value.length, next.value.length);
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
}
service.ts file:
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { environment } from '../../../environments/environment';
import { Observable } from 'rxjs';
import { UpdatePersonalProfileInformation, UpdateAddressProfileInformation } from '../../models/update-profile-info-model/update-profile-info.model';

@Injectable({
    providedIn: 'root'
})
export class UpdateNotaryProfileInformationService {
    private apiUrl = `${environment.apiurl}/api/InternalUser/UpdatePersonalInformation`;
    private apiUrlUpdateAddressInformation = `${environment.apiurl}/api/InternalUser/UpdateAddressInformation`;


    constructor(private http: HttpClient) { }

    /**
     * Sends the DTO to the backend.
     * Returns an Observable that emits `true` when the update succeeds.
     */
    updatePersonalInformation(payload: UpdatePersonalProfileInformation): Observable<boolean> {
        return this.http.post<boolean>(this.apiUrl, payload);
    }

    updateAddressInformation(payload: UpdateAddressProfileInformation[]): Observable<boolean> {
        return this.http.post<boolean>(this.apiUrlUpdateAddressInformation, payload);
    }
}
