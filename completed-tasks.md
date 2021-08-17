// Copyright (c) Microsoft Corporation.
// Licensed under the MIT license.

/**
 * @summary Demonstrates how to run a query against an Azure Log Analytics workspace.
 */

import { DefaultAzureCredential } from "@azure/identity";
import { Durations, LogsQueryClient, LogsTable, QueryLogsOptions, MetricsQueryClient, Metric } from "@azure/monitor-query";
import * as dotenv from "dotenv";
import { stringify } from "querystring";
dotenv.config();

const monitorWorkspaceId = process.env.MONITOR_WORKSPACE_ID!;
/**
* This is an app to study the Azure Monitor Query SDK for JS. The Azure Monitor Query SDK provides APIs to fetch two
* categories of data stored in Azure Monitor: Metrics and Logs. See README for more information on metrics and logs.
*
* In this study, you'll be given a set of tasks with instructions. Each task is a method that is called from the main
* method. Read the instructions in each function's documentation.
*
* The docs and README that are open in the browser may be used. They contain useful information to complete these 
* tasks. If you come across a concept that you're unfamiliar with, see if these docs can provide some information on 
* that topic. If you don't find anything useful, let me know and I'll provide guidance. Any feedback on the 
* documentation and the SDK is greatly appreciated.
*/
export async function main() {
    await task1();
    await task2();
    await task3();
    await task4();
    await task5();
    await task6();
}

/**
 * In this task, you'll be required to interact with the Azure Monitor service to identify the names of all the
 * metrics available for an Azure resource. The task is to print the names of these metrics.
 *
 * 1. Create a metrics client that authenticates using Azure Active Directory.
 * 2. Use the metrics client to get all the metric definitions available for a resource ID.
 *    2a. Use resource subscriptions/2cd617ea-1866-46b1-90e3-fffb087ebf9b/resourceGroups/metrics-advisor/providers/Microsoft.CognitiveServices/accounts/js-metrics-advisor
 *    2b. Print the list of all metric definition names.
 * 
 * Questions to ask user - Can you show how would you find resource id for this resource?
 * 
 * Questions to ask Scott and Srikanta - Should I ask the user to figure out where the resource URI is? So we can understand
 */
async function task1() {
    console.log("-------------------Executing task1-------------------");
    // your code here
    const tokenCredential = new DefaultAzureCredential();
    const metricsQueryClient = new MetricsQueryClient(tokenCredential);
    const metricsResourceId = "subscriptions/2cd617ea-1866-46b1-90e3-fffb087ebf9b/resourceGroups/metrics-advisor/providers/Microsoft.CognitiveServices/accounts/js-metrics-advisor";

    if (!metricsResourceId) {
        throw new Error("METRICS_RESOURCE_ID must be set in the environment for this sample");
    }

    const result = await metricsQueryClient.getMetricDefinitions(metricsResourceId);

    for (const definition of result.definitions) {
        console.log(`Definition = ${definition.name}`);
    }

    console.log("-------------------Completed task1-------------------");
}

/**
 * In this task, you'll fetch the time series data of one of the metrics that was listed in task 1. When you have
 * the results, print the time series data to console.
 *
 * 1. Use the metrics client from task 1 to query the "SuccessfulCalls" metric. You can copy the code over.
 *    1a. Use resource subscriptions/2cd617ea-1866-46b1-90e3-fffb087ebf9b/resourceGroups/metrics-advisor/providers/Microsoft.CognitiveServices/accounts/js-metrics-advisor
 *    1b. Print the resulting set of metric values along with the timestamp.
 * 2. You can set the timespan to last 30 days if you want to restrict the results. Use this for reference - https://en.wikipedia.org/wiki/ISO_8601#Time_intervals
 */
async function task2() {
    console.log("-------------------Executing task2-------------------");
    // your code here
    const tokenCredential = new DefaultAzureCredential();
    const metricsQueryClient = new MetricsQueryClient(tokenCredential);
    const metricsResourceId = "https://ms.portal.azure.com/#@microsoft.onmicrosoft.com/resource/subscriptions/2cd617ea-1866-46b1-90e3-fffb087ebf9b/resourceGroups/metrics-advisor/providers/Microsoft.CognitiveServices/accounts/js-metrics-advisor";

    const metricsResponse = await metricsQueryClient.queryMetrics(metricsResourceId,["SuccessfulCalls"]);

    console.log(
        `Query cost: ${metricsResponse.cost}, interval: ${metricsResponse.granularity}, time span: ${metricsResponse.timespan}`
    );

    const metrics: Metric[] = metricsResponse.metrics;
    console.log(`Metrics:`, JSON.stringify(metrics, undefined, 2));
    console.log("-------------------Completed task2-------------------");
}

/**
 *  In this task, you'll repeat most of what you did in task 2 but with some additional constraints on the
 * time series data you want to fetch.
 *
 * 1. Copy the code from task 2.
 * 2. Modify the query to get average and maximum aggregations.
 * 3. Set the metrics granularity interval to 10 minutes. Use this for reference - https://en.wikipedia.org/wiki/ISO_8601#Durations
 * 4. Print the resulting set of metric values (average and maximum only) along with the timestamp.
 * 5. Adjust the value of metrics granularity interval if needed.
 */
async function task3() {
    console.log("-------------------Executing task3-------------------");
    // your code here
    const tokenCredential = new DefaultAzureCredential();
    const metricsQueryClient = new MetricsQueryClient(tokenCredential);
    const metricsResourceId = "subscriptions/2cd617ea-1866-46b1-90e3-fffb087ebf9b/resourceGroups/metrics-advisor/providers/Microsoft.CognitiveServices/accounts/js-metrics-advisor";
    const metricsResponse = await metricsQueryClient.queryMetrics(metricsResourceId, ["SuccessfulCalls"],  {
        timespan:"P30D",
        aggregations: ["Average", "Maximum"],
        granularity: "PT1M"
    });

    console.log(
        `Query cost: ${metricsResponse.cost}, interval: ${metricsResponse.granularity}, time span: ${metricsResponse.timespan}`
    );

    const metrics: Metric[] = metricsResponse.metrics;
    console.log(JSON.stringify(metrics[0].timeseries, undefined, 2));
    console.log(`Metrics:`, JSON.stringify(metrics, undefined, 2));
    // Make the user print out all values of rows - dig into the timeseries data further
    console.log("-------------------Completed task3-------------------");
}

/**
 * In this task, you'll interact with Azure Monitor to fetch logs from an Azure Log Analytics workspace.
 * Use workspace ID "598029db-4756-4768-87f3-d6d45ed0ebd7" for the next 3 tasks.
 *
 * 1. Create a logs client that authenticates using Azure Active Directory.
 * 2. Use query "AppEvents | project TimeGenerated, Name, AppRoleInstance | order by TimeGenerated asc | limit 10".
 * 3. Set the time duration to last 30 days only. Use this for reference - https://en.wikipedia.org/wiki/ISO_8601#Time_intervals
 * 4. Print the log results to the console
*/
async function task4() {
    console.log("-------------------Executing task4-------------------");
    // your code here
    const tokenCredential = new DefaultAzureCredential();
    const logsQueryClient = new LogsQueryClient(tokenCredential);
    const kustoQuery =
        "AppEvents | project TimeGenerated, Name, AppRoleInstance | order by TimeGenerated asc | limit 10";

    console.log(`Running '${kustoQuery}' over the last 1 day`);

    const result = await logsQueryClient.queryLogs(
        monitorWorkspaceId,
        kustoQuery,
        Durations.lastDay
    );

    const tablesFromResult: LogsTable[] | undefined = result.tables;

    if (tablesFromResult == null) {
        console.log(`No results for query '${kustoQuery}'`);
        return;
    }

    console.log(
        `Results for query '${kustoQuery}'`
    );

    for (const table of tablesFromResult) {
        const columnHeaderString = table.columns
            .map((column) => `${column.name}(${column.type}) `)
            .join("| ");
        console.log("| " + columnHeaderString);

        for (const row of table.rows) {
            const columnValuesString = row.map((columnValue) => `'${columnValue}' `).join("| ");
            console.log("| " + columnValuesString);
        }
    }
    console.log("-------------------Completed task4-------------------");
}

/**
  * In this task, you'll fetch logs for a batch of queries through a single request.
  *
  * 1. Use the logs client from the previous task.
  * 2. Add the following queries to the batch request:
  *      "AppEvents | project TimeGenerated, Name, AppRoleInstance | order by TimeGenerated asc | limit 10"
  *      "AppEvents | count"
  *      "AppEvents | take 10"
  * 3. Set the timespan to last 30 days only. Use this for reference - https://en.wikipedia.org/wiki/ISO_8601#Time_intervals
  * 4. For each query in the batch, identify the corresponding response and print the query and its results.
*/
async function task5() {
    console.log("-------------------Executing task5-------------------");
    // your code here

    const tokenCredential = new DefaultAzureCredential();
    const logsQueryClient = new LogsQueryClient(tokenCredential);

    const queriesBatch = [
        {
            workspaceId: monitorWorkspaceId,
            query: "AppEvents | project TimeGenerated, Name, AppRoleInstance | order by TimeGenerated asc | limit 10",
            timespan: "P1D"
        },
        {
            workspaceId: monitorWorkspaceId,
            query: "AppEvents | count",
            timespan: "PT1D"
        },
        {
            workspaceId: monitorWorkspaceId,
            query:
                "AppEvents | take 10",
            timespan: "PT1D"
        },
    ];
    const result = await logsQueryClient.queryLogsBatch({
        queries: queriesBatch
    });

    let i = 0;
    for (const response of result.results!) {
        console.log(`Results for query with id: ${response.id}`);

        if (response.error) {
            console.log(` Query had errors:`, response.error);
        } else {
            if (response.tables == null) {
                console.log(`No results for query`);
            } else {
                console.log(
                    `Printing results from query '${queriesBatch[i].query}' for '${queriesBatch[i].timespan}'`
                );

                for (const table of response.tables) {
                    const columnHeaderString = table.columns
                        .map((column) => `${column.name}(${column.type}) `)
                        .join("| ");
                    console.log(columnHeaderString);

                    for (const row of table.rows) {
                        const columnValuesString = row.map((columnValue) => `'${columnValue}' `).join("| ");
                        console.log(columnValuesString);
                    }
                }
            }
        }
        // next query
        i++;
    }
    console.log("-------------------Completed task5-------------------");
}

/**
* In this task, you'll repeat most of what you did in task 4 but with some additional constraints on the
* time series data you want to fetch.
*
* Use workspace ID "598029db-4756-4768-87f3-d6d45ed0ebd7".
* Additional workspace ID is "640bfb1c-9109-4569-8a1b-4d9c92cc0eb2".
* 1. Copy the code from task 4.
* 3. Set one of the additionalWorkspaces as "640bfb1c-9109-4569-8a1b-4d9c92cc0eb2".
* 4. Print the log results to the console.
   */
async function task6() {
    console.log("-------------------Executing task6-------------------");
    // your code here

    const tokenCredential = new DefaultAzureCredential();
    const logsQueryClient = new LogsQueryClient(tokenCredential);
    //whatever query
    const kustoQuery =
        "AppEvents | project TimeGenerated, Name, AppRoleInstance | order by TimeGenerated asc | limit 10";

    console.log(`Running '${kustoQuery}' over the last 1 day`);

    const result = await logsQueryClient.queryLogs(
        monitorWorkspaceId,
        kustoQuery,
        "P1D",
        {
            additionalWorkspaces: ["640bfb1c-9109-4569-8a1b-4d9c92cc0eb2"]
        }
    );

    const tablesFromResult: LogsTable[] | undefined = result.tables;

    if (tablesFromResult == null) {
        console.log(`No results for query '${kustoQuery}'`);
        return;
    }

    console.log(
        `Results for query '${kustoQuery}'`
    );

    for (const table of tablesFromResult) {
        const columnHeaderString = table.columns
            .map((column) => `${column.name}(${column.type}) `)
            .join("| ");
        console.log("| " + columnHeaderString);

        for (const row of table.rows) {
            const columnValuesString = row.map((columnValue) => `'${columnValue}' `).join("| ");
            console.log("| " + columnValuesString);
        }
    }
    console.log("-------------------Completed task6-------------------");
}

main().catch((err) => {
    console.error("The sample encountered an error:", err);
    process.exit(1);
});
