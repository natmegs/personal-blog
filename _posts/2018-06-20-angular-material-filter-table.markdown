---
layout: post
title:  "Filterable Tables with Angular Material"
date:   2018-06-20 15:30:30 -0700
permalink: /blog/:title
comments: true
image: /assets/images/posts/With-Filters.png
---

The Angular Material [docs](https://material.angular.io/components/table/overview#filtering) more or less leave table filtering as an exercise for the reader. Here I'll walk through one possible solution for adding multiple column filters to a table using Angular Material.

###### Use Case

Picture this: You have a table with several columns. You have impatient users who do not (capital DO NOT) want to look through the table for the entry/entries that they need. They really (capital REALLY) need to be able to search on individual columns. But not just one column, all of them. This is what we're going for:

![Material Table with Filters](/assets/images/posts/With-Filters.png)


###### Implementation

This is what we're starting with:

![Material Table without Filters](/assets/images/posts/Table.png)

The table has been set up with Material table components, and the table data has been turned into a `MatTableDataSource`. This is important because this class contains filtering properties we will use when we add the filters to the tables.

```html
<table mat-table [dataSource]="dataSource">
  <ng-container matColumnDef="name">
    <th mat-header-cell *matHeaderCellDef>
      Tenant Name
    </th>
    <td mat-cell *matCellDef="let person">{{person.name}}</td>
  </ng-container>
  <ng-container matColumnDef="id">
    <th mat-header-cell *matHeaderCellDef>
      ID
    </th>
    <td mat-cell *matCellDef="let person">{{person.id}}</td>
  </ng-container>
  <ng-container matColumnDef="favouriteColour">
    <th mat-header-cell *matHeaderCellDef>Favourite Colour</th>
    <td mat-cell *matCellDef="let person">{{person.colour}}</td>
  </ng-container>
  <ng-container matColumnDef="pet">
    <th mat-header-cell *matHeaderCellDef>Pet</th>
    <td mat-cell *matCellDef="let person">{{person.pet}}</td>
  </ng-container>
  <tr mat-header-row *matHeaderRowDef="columnsToDisplay"></tr>
  <tr mat-row 
    *matRowDef="let person; columns: columnsToDisplay"></tr>
</table>
```

```typescript
import { Component, OnInit } from '@angular/core';
import { FormControl } from '@angular/forms';
import { MatTableDataSource } from '@angular/material';

@Component({
  selector: 'my-app',
  templateUrl: './app.component.html',
  styleUrls: [ './app.component.css' ]
})
export class AppComponent implements OnInit  {

  people = [
    {
      name: 'John',
      id: 1,
      colour: 'Green',
      pet: 'Dog'
    },
    {
      name: 'Sarah',
      id: 2,
      colour: 'Purple',
      pet: 'Cat'
    },
    {
      name: 'Lindsay',
      id: 3,
      colour: 'Blue',
      pet: 'Lizard'
    },
    {
      name: 'Megan',
      id: 4,
      colour: 'Orange',
      pet: 'Dog'
    }
  ];

  dataSource = new MatTableDataSource();
  columnsToDisplay = ['name', 'id', 'favouriteColour', 'pet'];

  constructor() {
    this.dataSource.data = this.people;
  }

  ngOnInit() {
    
  }
}
```

Now we add search inputs to the column headers, which the user will use to search by column.

```html
<table mat-table [dataSource]="dataSource">
  <ng-container matColumnDef="name">
    <th class="header" mat-header-cell *matHeaderCellDef>
      Tenant Name
      <mat-form-field class="filter" floatLabel="never">
        <mat-label>Search</mat-label>
        <input matInput [formControl]="nameFilter">
      </mat-form-field>
    </th>
    <td mat-cell *matCellDef="let person">{{person.name}}</td>
  </ng-container>
  <ng-container matColumnDef="id">
    <th class="header" mat-header-cell *matHeaderCellDef>
      ID
      <mat-form-field class="filter" floatLabel="never">
        <mat-label>Search</mat-label>
        <input matInput [formControl]="idFilter">
      </mat-form-field>
    </th>
    <td mat-cell *matCellDef="let person">{{person.id}}</td>
  </ng-container>
  <ng-container matColumnDef="favouriteColour">
    <th class="header" mat-header-cell *matHeaderCellDef>
      Favourite Colour
      <mat-form-field class="filter" floatLabel="never">
        <mat-label>Search</mat-label>
        <input matInput [formControl]="colourFilter">
      </mat-form-field>
    </th>
    <td mat-cell *matCellDef="let person">{{person.colour}}</td>
  </ng-container>
  <ng-container matColumnDef="pet">
    <th class="header" mat-header-cell *matHeaderCellDef>
      Pet
      <mat-form-field class="filter" floatLabel="never">
        <mat-label>Search</mat-label>
        <input matInput [formControl]="petFilter">
      </mat-form-field>
    </th>
    <td mat-cell *matCellDef="let person">{{person.pet}}</td>
  </ng-container>
  <tr mat-header-row *matHeaderRowDef="columnsToDisplay"></tr>
  <tr mat-row 
    *matRowDef="let person; columns: columnsToDisplay"></tr>
</table>
```

And we connect the inputs to `FormControl`s from the `ReactiveFormsModule`. The component is modified to add:

```typescript
nameFilter = new FormControl('');
idFilter = new FormControl('');
colourFilter = new FormControl('');
petFilter = new FormControl('');
```

Great, now we have a table with a `DataSource` and search filter inputs on each column, but the filters don't do anything. To get the table to show only results that match the filters, we will use the `filter` and `filterPredicate` properties of `MatTableDataSource`. The `filter` property is a string, and the `filterPredicate` is a function that takes a data object and a filter to determine if the data object is considered a match.

We will add an object that will represent the values of the column filters to the component: 

```typescript
filterValues = {
    name: '',
    id: '',
    colour: '',
    pet: ''
};
```

And we will watch the value of the filter inputs and modify this filter object and the data source's filter property when they change. We must assign the stringified version of the filter object to the data source's `filter` property (remember, the filter property is a string):

```typescript
  ngOnInit() {
    this.nameFilter.valueChanges
      .subscribe(
        name => {
          this.filterValues.name = name;
          this.dataSource.filter = JSON.stringify(this.filterValues);
        }
      )
    this.idFilter.valueChanges
      .subscribe(
        id => {
          this.filterValues.id = id;
          this.dataSource.filter = JSON.stringify(this.filterValues);
        }
      )
    this.colourFilter.valueChanges
      .subscribe(
        colour => {
          this.filterValues.colour = colour;
          this.dataSource.filter = JSON.stringify(this.filterValues);
        }
      )
    this.petFilter.valueChanges
      .subscribe(
        pet => {
          this.filterValues.pet = pet;
          this.dataSource.filter = JSON.stringify(this.filterValues);
        }
      )
  }
```

And... still doesn't work. We have to change the data source's `filterPredicate` to tell it how to interpret the filter information.

```typescript
  constructor() {
    this.dataSource.data = this.people;
    this.dataSource.filterPredicate = this.tableFilter();
  }

  tableFilter(): (data: any, filter: string) => boolean {
    let filterFunction = function(data, filter): boolean {
      let searchTerms = JSON.parse(filter);
      return data.name.toLowerCase().indexOf(searchTerms.name) !== -1
        && data.id.toString().toLowerCase().indexOf(searchTerms.id) !== -1
        && data.colour.toLowerCase().indexOf(searchTerms.colour) !== -1
        && data.pet.toLowerCase().indexOf(searchTerms.pet) !== -1;
    }
    return filterFunction;
  } 
```

And now we have working filters on each column. You can try it out on [Stackblitz](https://stackblitz.com/edit/angular-f3mmmp).