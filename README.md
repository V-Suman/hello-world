Bug: The displayed approval date is one day prior. 
Details: When the user searches for notary records with an approval date from and approval date to.. the results are displayed
and the results contain incorrect date on the UI. 
request object: dateRangeSearch {
    "approvalDateFrom": "2007-10-24",
    "approvalDateTo": "2007-10-24",
    "searchType": 3
}
response array's one record: {
    "applicantId": 14845,
    "newRenewal": "Renew",
    "lastName": "Adrian",
    "firstName": "Norman",
    "middleName": "J.",
    "cityTown": "Carlisle",
    "dateOfBirth": "1940-06-02",
    "county": "Middlesex",
    "approvalDate": "2007-10-24",
    "createdDate": "2007-10-16",
    "isRemoteNotary": false
}
I attached the picture of how it is looking on the UI.. 
Odd thing: The odd thing I observed here is.. when the search results appear with incorrect approval date.. and I navigate to 
a notary's record by clicking the notary ID.. and come back to the current page.. the search grid displays correct data? This 
is very strange to me. I added that picture as well.
notary-records.component.html:
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
            <label for="notaryId">Notary ID #</label>
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
notary-records.component.ts file: 
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
          console.log(response);
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
