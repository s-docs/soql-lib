# Query Selector (QS)

![Deploy to Scratch Org and run tests](https://github.com/beyond-the-cloud-dev/query-selector/actions/workflows/ci.yml/badge.svg)

[![codecov](https://codecov.io/gh/beyond-the-cloud-dev/query-selector/branch/main/graph/badge.svg)](https://codecov.io/gh/beyond-the-cloud-dev/query-selector)

----


Apex QS provides functional constructs for SOQL.

## Examples

```apex
//SELECT Id FROM Account
List<Account> accounts = (List<Account>) QS.of(Account.sObjectType).asList();
```


Find more examples in the [documentation](https://btc-query-selector.vercel.app/).

## Deploy to Salesforce

<a href="https://githubsfdeploy.herokuapp.com?owner=beyond-the-cloud-dev&repo=query-selector&ref=main">
  <img alt="Deploy to Salesforce"
       src="https://raw.githubusercontent.com/afawcett/githubsfdeploy/master/deploy.png">
</a>

## Guidelines

1. Selector class should be small and contains only selector base configuration + very generic methods. Selector should allow building SOQL inline in the place of usage.

We should avoid constructions like that:

```apex
public with sharing AccountSelector extends fflib_SObjectSelector {
   public Schema.SObjectType getSObjectType(){
      return Account.sObjectType;
   }

   public override List<Schema.SObjectField> getSObjectFieldList(){
      return new List<Schema.SObjectField> {
         Account.Id,
         Account.Name
      };
   }

   public List<Account> selectById(Set<Id> recordIds){
      fflib_QueryFactory query = newQueryFactory();

      fflib_SObjectSelector addressesSelector = new AddressesSelector();
      addressesSelector.configureQueryFactoryFields(query, 'InvoiceAddress__r');

      return (List<Account>) Database.query( query.toSOQL() );
   }

   // 500 more lines here
}
```

2. Developers should be able to modify SOQL with fields, conditions, and other clauses as needed.
3. Developers should not spend too much time on selector methods naming convention.
- How to name a method with 3 conditions? selectBySomethingAAndSomethingBAndSomehtingC?
- What if an order should be passed as an argument?

```java
public with sharing AccountSelector extends fflib_SObjectSelector {
   public Schema.SObjectType getSObjectType(){
      return Account.sObjectType;
   }

   public override List<Schema.SObjectField> getSObjectFieldList(){
      return new List<Schema.SObjectField> {
         Account.Id,
         Account.Name
      };
   }

   public List<Account> selectById(Set<Id> recordIds){
      fflib_QueryFactory query = newQueryFactory();

      fflib_SObjectSelector addressesSelector = new AddressesSelector();
      addressesSelector.configureQueryFactoryFields(query, 'InvoiceAddress__r');

      return (List<Account>) Database.query( query.toSOQL() );
   }

   // selectBySomethingElse
   // selectBySomethingElseAndMore
   // selectBySomethingElseButWithLimit
}
```

4. Avoid methods where the only difference is a list of fields, limit, order, etc.
5. Selector framework should allow controlling FLS and sharing.
- If fflib_SObjectSelector is marked as `with sharing` it is applied to all methods. However, there are some cases where some methods should be `without sharing`? What we can do? A new selector, but `without sharing`? Not good enough.
6. Selector framework should be able to do dynamic binding without constructions as below:

```
public List<Account> selectByName(Set<String> names){
    fflib_QueryFactory query = newQueryFactory();
    query.setCondition('Name IN :names');
    return (List<Account>) Database.query( query.toSOQL() );
}
```

7. Selector framework should allow for mocking in unit tests.
8. Selector framework should allow building query/adjust on the fly.
----

## License notes

- For proper license management each repository should contain LICENSE file similar to this one.
- each original class should contain copyright mark: © Copyright 2022, Beyond The Cloud Dev Authors
