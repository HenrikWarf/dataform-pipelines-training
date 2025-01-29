# BigQuery Pipelines (Dataform) Training Guide

This guide provides a concise overview of key Dataform concepts and features, empowering data teams to effectively develop, version control, and orchestrate SQL workflows in BigQuery. Each section includes a link to the Dataform documentation for in-depth learning. Also we have added a hands on lab to help you understand the concepts better which also included the use of LLM to generate the content directly in BigQuery via Dataform.

We will be adding video content to this guide to help you understand the concepts better [coming soon].

## Video Content: 

### Dataform Introduction

[video coming soon] 

### Dataform repository and workspace

[video coming soon] 

### Dataform workflow development

[video coming soon] 

### Dataform workflow compilation and execution

[video coming soon] 

### Dataform data quality testing

[video coming soon] 

## 1. Introduction to Dataform

Dataform is an open-source framework for managing and orchestrating SQL-based transformations in BigQuery. It simplifies the process of building, testing, and deploying data pipelines by providing a framework for managing SQL-based transformations.

**Key Features and Benefits:**

*   **Modular Data Transformations:** Organize and build complex data pipelines using modular and reusable SQL-based models.
*   **Version Control and Collaboration:** Integrate with Git for seamless version control, enabling collaboration and easy tracking of changes.
*   **Automated Code Validation:** Validates your SQL code and provides warnings if any issues are detected.
*   **Dependency Management:** Define dependencies between models and tasks, ensuring the correct execution order.
*   **Improved Data Pipeline Maintainability:** Simplifies the management and maintenance of data pipelines.

For a comprehensive overview of Dataform, refer to the Dataform documentation: [Dataform Overview](https://cloud.google.com/dataform/docs/overview)

## 2. Repositories and Workspaces

In Dataform, each project is stored in a repository, which houses configuration files, SQLX files, and JavaScript files. These repositories support version control and can be linked to a Git provider like GitHub or GitLab. Currently, the only available options for Git providers are GitHub and GitLab.

Dataform also allows you to create workspaces within a repository for development. Workspaces provide an environment to develop and test Dataform code before deploying it to production.

**Key Concepts:**

*   **Repository:** A collection of files that define your Dataform project.
*   **Workspace:** An isolated environment for developing and testing Dataform code.
*   **Version Control:** Using Git to track changes and manage file versions.

Learn more about repositories and workspaces in the Dataform documentation: [Create Repository](https://cloud.google.com/dataform/docs/create-repository)

## 3. Workflow Development

Dataform provides a structured approach to developing data transformation workflows. You can define various actions within a workspace, including:

*   **Source data declarations:** Declare the source data for your transformations. This involves specifying the location and format of your raw data, allowing Dataform to access and utilize it in your workflows.

    ```sql
    config {
      type: "declaration",
      schema: "my_dataset",
      name: "my_table",
      database: "bigquery",
    }
    ```

*   **Tables and views:** Define tables and views using SQLX files. SQLX files extend SQL with Dataform-specific features, enabling you to define table schemas, transformations, and dependencies.

    ```sql
    config {
      type: "table",
      schema: "my_dataset",
      name: "my_transformed_table"
    }

    SELECT
      *
    FROM
      ${ref('my_table')}
    ```

*   **Incremental tables:** Create tables that are updated incrementally. This is useful for large datasets where only a subset of data changes frequently. Dataform efficiently updates these tables by processing only the changes, saving time and resources.

    ```sql
    config {
      type: "incremental",
      schema: "my_dataset",
      name: "my_incremental_table",
      uniqueKey: ["customer_id", "order_date"],
      partitionBy: {
        field: "order_date",
        data_type: "timestamp",
        granularity: "day"
      }
    }

    SELECT
      customer_id,
      order_date,
      -- ... other columns
    FROM
      ${ref('my_table')}
    WHERE
      ${incremental()}    -- this function automatically generates the incremental where clause
    ```

*   **Dependencies:** Specify dependencies between actions to ensure correct execution order. Dataform automatically manages these dependencies, ensuring that tasks are executed in the correct sequence.

    ```sql
    config {
      type: "table",
      schema: "my_dataset",
      name: "my_final_table",
      dependencies: [
        "my_transformed_table",
        "my_incremental_table"
      ]
    }

    SELECT
      *
    FROM
      ${ref('my_transformed_table')}
    JOIN
      ${ref('my_incremental_table')} USING (customer_id)
    ```

*   **Documentation:** Add documentation to tables and columns within your code. This documentation is integrated with BigQuery, making it easily accessible and searchable.

    ```sql
    config {
      type: "table",
      database: "your_project_id",
      schema: "ILS",
      name: "test_david_ILS_categories",
      description: "A table description here",
      columns: {
        category_id: {
          description: "Description of category_id",
          type: "STRING"
        },
        category_name: {
          description: "Description of category_name",
          type: "STRING"
        }
      }
    }
    ```

*   **Data quality tests:** Implement assertions to ensure data consistency. Assertions allow you to define rules and checks on your data, ensuring that it meets your quality standards.

    ```sql
    config {
      type: "table",
      name: "my_table",
      assertions: {
        uniqueKey: "my_column",
        notNull: "another_column"
      }
    }
    ```

Dataform uses Dataform core, an extension of SQL, to define these actions. Dataform core provides features like dependency management, automated data quality testing, and data documentation.

**Code Reuse with JavaScript**

Dataform allows you to use JavaScript for code reuse across files and repositories. You can create JavaScript files inside the `includes` folder to define variables and functions that can be accessed across multiple files within a Dataform repository.

To achieve code reuse across multiple Dataform repositories, you can leverage packages. Packages are collections of reusable code that can be created and imported into your Dataform projects.

It's important to highlight that the documentation created in Dataform is directly pushed to BigQuery. This seamless integration allows for easy access to documentation within BigQuery and facilitates the use of other BigQuery tools for data discovery and exploration.

For detailed information on workflow development, refer to the Dataform documentation: [SQL Workflows](https://cloud.google.com/dataform/docs/sql-workflows)

## 4. Workflow Compilation and Execution

Dataform compiles the workflow code in your workspace in real-time, translating Dataform core into SQL queries. You can view the compiled queries and details of each action in the workspace.

Once the code is compiled, you can execute the workflow. Dataform executes the actions in the dependency tree, ensuring that tasks are executed in the correct order.

**Scheduling Options:**

Dataform offers various options for scheduling workflow executions:

*   Workflow configurations: Schedule executions directly within Dataform.
*   Workflows and Cloud Scheduler: Use Cloud Scheduler to trigger Dataform workflows.
*   Cloud Composer: Orchestrate Dataform workflows using Cloud Composer.

Learn more about workflow compilation and execution in the Dataform documentation: [Code Lifecycle](https://cloud.google.com/dataform/docs/code-lifecycle)

Learn more about scheduling options in the Dataform documentation: [Dataform Scheduling](https://cloud.google.com/dataform/docs/workflow-configurations)

## 5. Data Quality Testing

Dataform allows you to implement data quality tests, called assertions, to ensure the consistency and reliability of your data. You can define assertions within the `config` block of a SQLX file or in separate assertion files.

**Types of Assertions:**

| Assertion Type      | Description                                                                 |
| ------------------- | --------------------------------------------------------------------------- |
| Uniqueness          | Ensures that a column contains only unique values.                           |
| NotNull             | Checks if a column contains any null values.                                |
| GreaterThan         | Verifies that values in a column are greater than a specified value.        |
| Referential Integrity | Ensures that relationships between tables are maintained.                    |

**Example of Assertions:**

```sql
config {
  type: "table",
  name: "my_table",
  assertions: {
    uniqueKey: "my_column",
    notNull: "another_column"
  }
}
```

For more information on data quality testing in Dataform, refer to the Dataform documentation: [Dataform Overview](https://cloud.google.com/dataform/docs/overview)

## 6. Resources

Dataform provides various resources to help users understand and utilize its features effectively. These resources include:

*   **Pricing:** Information on Dataform pricing and usage costs.
*   **Quotas and limits:** Details about the quotas and limits associated with Dataform usage.
*   **Locations:** Information on the available regions and locations for Dataform deployments.
*   **Support:** Access to support channels and resources for assistance with Dataform.

You can find these resources in the Dataform documentation: [Dataform Docs](https://cloud.google.com/dataform/docs)

## 7. Summary

Dataform offers a robust and efficient way to manage data transformation workflows in BigQuery. By leveraging its 
features such as modularity, version control, automated testing, and dependency management, data teams can improve the quality, maintainability, and reliability of their data pipelines. This guide has covered the essential aspects of Dataform, and the links to the Dataform documentation provide further details for those who want to explore specific topics in greater depth.
