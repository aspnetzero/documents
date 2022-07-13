# Edit Mode For Phone Numbers

Final UI is shown below:

<img src="images/phone-book-angular-show-phones-in-ui.png" alt="Phone book edit mode" class="img-thumbnail" />

## Service Proxy Generation

First, run (prefer Ctrl+F5 to be faster) the server side application (.Web.Host project). Then run **nswag/refresh.bat** file on the client side to re-generate service proxy classes (they are used to call server side service methods).

Since we added a new service, we should add it to **src/shared/service-proxies/service-proxy.module.ts**. Just open it and add **ApiServiceProxies.PersonServiceProxy** to the providers array. This step is only required when we add a new service. If we change an existing service, it's not needed.

## View

Add phone numbers to the table.

```html
 <p-table...>
	<ng-template pTemplate="header">
		<tr>
			<th>
				{{ 'PhoneNumbers' | localize }}
			</th>
		</tr>
	</ng-template>
	<ng-template pTemplate="body" let-record="$implicit">
		<tr>
			<td>
				<span class="p-column-title">{{ 'PhoneNumbers' | localize }}</span>
				<table>
					<tbody>
                        <tr *ngFor="let phone of record.phones">
                            <td>{{getPhoneTypeString(phone.type)}}</td>
                            <td>{{phone.number}}</td>
                        </tr>
					</tbody>
				</table>
			</td>
		</tr>
	</ng-template>
</p-table>
```

## PhoneBook Component Class

Add **getPhoneTypeString** method.

```typescript
getPhoneTypeString(phoneType: PhoneType): string {
    switch (phoneType) {
        case 1:
            return this.l('Home');
        case 2:
            return this.l('Business');
        default:
            return this.l('Mobile');
    }
}
```

## Next

- [Edit Mode For Phone Numbers](Developing-Step-By-Step-Angular-Edit-Mode-Phone-Numbers)
