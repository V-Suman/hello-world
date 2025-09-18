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
notary-records.component.html file: 
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
                           placeholder="Enter First Name"
                           class="custom-input">
            </kendo-textbox>
          </kendo-formfield>

          <kendo-formfield>
            <label for="lastName">Last Name</label>
            <kendo-textbox id="lastName"
                           [(ngModel)]="lastName"
                           name="lastName"
                           placeholder="Enter Last Name"
                           class="custom-input">
            </kendo-textbox>
          </kendo-formfield>

          <kendo-formfield>
            <label for="cityTown">City / Town</label>
            <kendo-textbox id="cityTown"
                           [(ngModel)]="cityTown"
                           name="cityTown"
                           placeholder="Enter City/Town"
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
            <label for="notaryId">Full Notary ID </label>
            <kendo-textbox id="notaryId"
                           [(ngModel)]="notaryId"
                           name="notaryId"
                           placeholder="Enter Notary ID"
                           class="custom-input"
                           
                           (keypress)="allowOnlyNumbers($event)"
                           (input)="onNotaryIdInput($event)"
                           >
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
notary-record.component.ts file:
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
        console.log(wrapper);

        this.search.searchNotariesWithDateRange(wrapper)
            .subscribe(response => {
                const raw = response.notarySearchResultsInternalDto;
                this.searchResults = raw.map((item: any) => ({
                    ...item,
                    approvalDate: item.approvalDate
                        ? this.parseLocalDate(item.approvalDate)  
                        : null,
                    createdDate: item.createdDate
                        ? this.parseLocalDate(item.createdDate)   
                        : null
                }));

                this.displayMessage = response.message;
                this.pageSize = response.length;
                this.toggleSearchGrid = true;
                this.searchStateData.lastCriteria = wrapper;
                this.searchStateData.lastResults = response.notarySearchResultsInternalDto?.slice();
            },
                err => console.error(err));
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

  // Validates Notary ID on blur. Clears the field and sets an error.
  validateNotaryId(): void {
    if (this.notaryId) {
      this.notaryIdError = "Notary ID search parameter cannot be empty";
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
notary-records.component.css file: 
:host {
  display: flex;
  flex-direction: column;
}

kendo-textbox, kendo-datepicker{
  --kendo-color-subtle: #aaa !important;
}

kendo-expansionpanel {
  width: 640px;
}

.k-expander:not(.k-expanded) + .k-expander:not(.k-expanded) {
  border-top-width: 1px !important;
}

.page-wrapper {
  display: flex;
  flex-direction: row;
  column-gap: 2rem;
  margin-left: 3rem;
}

.individual-search-instructions{
    display:flex;
    flex-direction: column;
    row-gap:1rem;
    margin-bottom:1.4rem;
}

.search-instructions{
    padding-top: 1rem;
}

.instructions-and-search-wrapper {
  display: flex;
  flex-direction: column;
  row-gap: 2rem;
  margin-bottom: 6rem;
}

strong{
    font-size:15px;
}
.notary-title {
  color: #074e72;
  margin-bottom: 0!important
}

.card-title {
  margin-top: 0 !important;
  color: white;
  margin-bottom: 0 !important
}

label {
    font-weight:bold;
}

.k-form-field {
  display: flex;
  flex-flow: column;
  gap: 0.5rem;
  align-self: flex-start;
}

.k-form-field-wrap{
    margin-top:1rem!important;
}

.notary-form {
  display: flex;
  flex-direction: column;
  gap: 2.5rem;
}

.form-row {
  display: flex;
  gap: 1rem;
}

/* Notary Category styling */
.notary-category-group {
  display: flex;
  flex-direction: column;
}

.k-checkbox-wrap {
  margin-left: 4rem !important;
}

.remoteNotariesLabel{
    margin-bottom:0.8rem;
}

/* Approval row styling */
.approval-row {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 1rem;
}

.or-separator {
  font-weight: bold;
  margin-top: 1.7rem;
}

.approval-range-group {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.date-range-special{
    display:flex!important;
    flex-direction:column!important;
}

.radio-buttons-group{
    display:flex;
    flex-direction:row;
    justify-content:space-around;
}

.radio-buttons-special {
  display: flex !important;
  flex-direction: column;
  align-items: center;
  gap: 0;
}

.dash {
  margin-right: 0.7rem;
  margin-top: 1.3rem;
  margin-left: 0.6rem;
  font-size: 2rem;
}

/* Buttons styling */
.button-row {
  display: flex;
  gap: 1rem;
  margin-top:1rem;
}

.search-button {
  background-color: #074e72;
  border-color: #074e72;
}

::ng-deep .custom-input {
  width: 12rem;
  height: 2.2rem;
  border-radius: 0.36rem;
  border: 1px solid #adadad;
  box-sizing: border-box;
}

.no-label-compensation{
    margin-top:1.6rem;
}

button {
    width:7rem;
    height:2rem;
}

.custom-button-alt {
  border-color: #074e72;
  background-color: transparent;
  color: #074e72;
}

.search-results-grid {
  margin-top: 1.6rem;
  width: 71rem;
  margin-bottom: 5rem;
}

#notaryId{
    margin-bottom:2rem;
}

.custom-grid .k-grid-header {
  background-color: #074e72 !important;
}

  .custom-grid .k-grid-header .k-header,
  .custom-grid .k-grid-header .k-link {
    color: #ffffff !important;
  }

/* Row Hover Styling */
.custom-grid .k-grid-content tr.k-master-row:hover {
  background-color: #f1f5f9;
}

.custom-grid .k-pager-wrap {
  background-color: #ffffff;
  border-top: 1px solid #ddd;
}

::ng-deep thead {
  background-color: #074e72!important;
  color: white!important;
}

.matched-records-text {
  color: #074e72;
  font-weight:bold;
  margin-bottom:1rem;
}

.form-row kendo-formfield {
  position: relative;
}

/* Style the error message so it doesn't push content around */
.error-message {
  position: absolute;
  top: 4.1rem;
  left: 0;
  color: red;
  font-size: 0.85rem; 
  pointer-events: none;
}

.type-options {
  display: flex;
  gap: 3rem;
  align-self:flex-start;
}

.type-option {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.type-option label {
  margin-top: 0.5rem;
  font-weight: bold;
}

.date-range-and-dash {
  display: flex;
  align-items: center;
  align-self: flex-start;
}

#type-new, #type-both {
    margin-left: 5px;
}

#type-renewed {
    margin-left: 19px;
}

.notary-id-link, .notary-id-link:active{
    color:#074e72;
    font-weight:600;
}

.notary-id-link:hover{
    text-decoration:underline;
}
