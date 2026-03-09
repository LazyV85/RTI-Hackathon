# Take Control

## Backstory
Fabrikam's immediate requirement is to control the spread of bad reputation through defective production and shipping these defective products to their customers. For this, they need your help to work on previously gathered data and make actionable insights available to all the management and operational teams.

## Setup
For running this Act, 
1. You will need knowledge of Kusto Query Language (KQL).
2. You will need to access to create a Fabric Activator and Real-Time Dashboard.

## Challenges 

## 1. Get Latest Shipping details
<details>
<summary>Hint!</summary>
Use update policy to parse xml into silver table.

Use Materialized View to get latest status details for each OrderNumber
<details>
<summary>Guide</summary>
Start by writing a query that gives the desired output from the source table.(KQL has a built-in function to parse xml)

Creating a new column that contains the output of a parse xml is recommended.

Then create the target table based on the query like shown in the example below.

```KQL
.set <target table> <|
<source table>
| <conditions>(where/extend)
| <output>(project/project-away/project-reorder)
```

The following codesnippet shows an example of how to create a fuction/updatepolicy in a KQL database.

```KQL
.create function
 with (docstring = '<Function description>', folder = 'UpdatePolicyFunctions')
     <function name>()
    {
    <source table>
    | <conditions>(where/extend)
    | <output>(project/project-away/project-reorder)
}

.alter table <target table> policy update
@'[{ "IsEnabled": true, "Source": "<source table>", "Query": "<function name>()", "IsTransactional": false, "PropagateIngestionProperties": true}]'
```

The following codesnippet shows how to create a materialized view that shows the latest row based on a group by and order by column.

Materialized views require summarize as the last output and this line cannot be changed after the view is created.

The backfill option creates a view on all the data on the source table. Without this option only new rows will be added to the materialized view.

The async option creates a background operation that prevents the creation from failing if the source data is to large(over 500.000 rows or 64MB of data).

```KQL
.create async materialized-view with(backfill=true) <Materialized view name> on table <Source table>{
<Source table>
| <conditions>(where/extend)
| summarize arg_max(<Order by column>, <column(s) to include>) by <Group by column>
}
```
</details>
</details>

## 2. Stop the shipping providers  and production line that is carrying the highest defect probability product for every 1 hour
<details>
<summary>Hint!</summary>

Use a combination of the following steps to achieve this

1. Join between product, operators, shipment, shipping provider, and production

2. Create 1 Alert to stop shipping provider with details of OrderNumber, ProductName, ShippingProvider, PhoneNumber of the provider, HQaddress

3. Create one more Alert to stop production line with details of ProductId, ProductName, OperatorId, OperatorName, PhoneNumber of the operator, AssetId, BatchId

4. Activator limiting to 5 details? Try combining ProductId, ProductName and OperatorId, Operatorname in KQL query into one column Ex: [OperatorId - OperatorName]

</details>

## 3. Create an operational dashboard for Fabrikam management
1. Show  all operators details.
2. Correlation between defective and non-defective products in 1 hour bins
4. Current shipment status (count of orders by status). 
5. Map showing shipment count by destination

<details>
<summary>Hint!</summary>

Use copilot dashboard creation from Real-Time Hub

Use the anomaly flag column you defined in the SQL Code Operator before, to see the defection & non-defective products and use that in a line chart
</details>

* Below is just an example of an operational dashboard for Fabrikam management, feel free to make this realtime dashboard your own.

<img src="../Assets/OperationalDashboard_Fabrikam.png" alt="Alt text" width="1000"/>



