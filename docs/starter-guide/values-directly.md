# 直接在具有Observables的模板中使用值

如果要在模板中直接使用值，请使用`AsyncPipe`

````js
this.accounts = this.restangular.all('accounts').getList();
````

````html
<tr *ngFor="let account of accounts | async">
  <td>{{account.fullName}}</td>
</tr>
````