# [Apache Airflow](https://airflow.apache.org/)
**Apache Airflow** is an open-source workflow management platform for data engineering pipelines. It started at Airbnb in October 2014 as a solution to manage the company's increasingly complex workflows. Creating Airflow allowed Airbnb to programmatically author and schedule their workflows and monitor them via the built-in Airflow user interface. From the beginning, the project was made open source, becoming an Apache Incubator project in March 2016 and a top-level Apache Software Foundation project in January 2019.

Airflow is written in Python, and workflows are created via Python scripts. Airflow is designed under the principle of "configuration as code". While other "configuration as code" workflow platforms exist using markup languages like XML, using Python allows developers to import libraries and classes to help them create their workflows.

## Overview
Airflow uses directed acyclic graphs (**DAGs**) to manage workflow orchestration. Tasks and dependencies are defined in Python and then Airflow manages the scheduling and execution. DAGs can be run either on a defined schedule (e.g. hourly or daily) or based on external event triggers (e.g. a file appearing in Hive). Previous DAG-based schedulers like Oozie and Azkaban tended to rely on multiple configuration files and file system trees to create a DAG, whereas in Airflow, DAGs can often be written in one Python file.

Airflow is written in Python, and workflows are created via Python scripts. Airflow is designed under the principle of "configuration as code". While other "configuration as code" workflow platforms exist using markup languages like XML, using Python allows developers to import libraries and classes to help them create their workflows.

## Managed providers
Three notable providers offer ancillary services around the core open source project.

- Astronomer has built a SaaS tool and Kubernetes-deployable Airflow stack that assists with monitoring, alerting, devops, and cluster management.
- Cloud Composer is a managed version of Airflow that runs on Google Cloud Platform (GCP) and integrates well with other GCP services.
- Amazon Web Services offers Managed Workflows for Apache Airflow starting from November 2020.

## Core Concepts
### Architecture Overview
Airflow is a platform that lets you build and run workflows. A workflow is represented as a DAG (a Directed Acyclic Graph), and contains individual pieces of work called Tasks, arranged with dependencies and data flows taken into account.

![work-flows.png](https://airflow.apache.org/docs/apache-airflow/stable/_images/edge_label_example.png)

A DAG specifies the dependencies between tasks, which defines the order in which to execute the tasks. Tasks describe what to do, be it fetching data, running analysis, triggering other systems, or more.

Airflow itself is agnostic to what you’re running - it will happily orchestrate and run anything, either with high-level support from one of our providers, or directly as a command using the shell or Python Operators.

#### Airflow components
Airflow’s architecture consists of multiple components. The following sections describe each component’s function and whether they’re required for a bare-minimum Airflow installation, or an optional component to achieve better Airflow extensibility, performance, and scalability.

##### Required components
A minimal Airflow installation consists of the following components:

- A scheduler, which handles both triggering scheduled workflows, and submitting Tasks to the executor to run. The executor, is a configuration property of the scheduler, not a separate component and runs within the scheduler process. There are several executors available out of the box, and you can also write your own.

- A webserver, which presents a handy user interface to inspect, trigger and debug the behaviour of DAGs and tasks.

- A folder of DAG files, which is read by the scheduler to figure out what tasks to run and when to run them.

- A metadata database, which airflow components use to store state of workflows and tasks. Setting up a metadata database is described in Set up a Database Backend and is required for Airflow to work.

##### Optional components
Some Airflow components are optional and can enable better extensibility, scalability, and performance in your Airflow:

- Optional worker, which executes the tasks given to it by the scheduler. In the basic installation worker might be part of the scheduler not a separate component. It can be run as a long running process in the CeleryExecutor, or as a POD in the KubernetesExecutor.

- Optional triggerer, which executes deferred tasks in an asyncio event loop. In basic installation where deferred tasks are not used, a triggerer is not necessary. More about deferring tasks can be found in Deferrable Operators & Triggers.

- Optional dag processor, which parses DAG files and serializes them into the metadata database. By default, the dag processor process is part of the scheduler, but it can be run as a separate component for scalability and security reasons. If dag processor is present scheduler does not need to read the DAG files directly. More about processing DAG files can be found in DAG File Processing

- Optional folder of plugins. Plugins are a way to extend Airflow’s functionality (similar to installed packages). Plugins are read by the scheduler, dag processor, triggerer and webserver. More about plugins can be found in Plugins.

#### Architecture Diagrams
The diagrams below show different ways to deploy Airflow - gradually from the simple “one machine” and single person deployment, to a more complex deployment with separate components, separate user roles and finally with more isolated security perimeters.

The meaning of the different connection types in the diagrams below is as follows:

- **brown solid lines** represent DAG files submission and synchronization

- **blue solid lines** represent deploying and accessing installed packages and plugins

- **black dashed lines** represent control flow of workers by the scheduler (via executor)

- **black solid lines** represent accessing the UI to manage execution of the workflows

- **red dashed lines** represent accessing the metadata database by all components

##### Basic Airflow deployment
This is the simplest deployment of Airflow, usually operated and managed on a single machine. Such a deployment usually uses the LocalExecutor, where the scheduler and the workers are in the same Python process and the DAG files are read directly from the local filesystem by the scheduler. The webserver runs on the same machine as the scheduler. There is no triggerer component, which means that task deferral is not possible.

Such an installation typically does not separate user roles - deployment, configuration, operation, authoring and maintenance are all done by the same person and there are no security perimeters between the components.

![basic-architecture.png](https://airflow.apache.org/docs/apache-airflow/stable/_images/diagram_basic_airflow_architecture.png)

##### Distributed Airflow architecture
This is the architecture of Airflow where components of Airflow are distributed among multiple machines and where various roles of users are introduced - Deployment Manager, DAG author, Operations User. You can read more about those various roles in the Airflow Security Model.

In the case of a distributed deployment, it is important to consider the security aspects of the components. The webserver does not have access to the DAG files directly. The code in the Code tab of the UI is read from the metadata database. The webserver cannot execute any code submitted by the DAG author. It can only execute code that is installed as an installed package or plugin by the Deployment Manager. The Operations User only has access to the UI and can only trigger DAGs and tasks, but cannot author DAGs.

The DAG files need to be synchronized between all the components that use them - scheduler, triggerer and workers. The DAG files can be synchronized by various mechanisms - typical ways how DAGs can be synchronized are described in Manage DAGs files of our Helm Chart documentation. Helm chart is one of the ways how to deploy Airflow in K8S cluster.

![distributed-architecture.png](https://airflow.apache.org/docs/apache-airflow/stable/_images/diagram_distributed_airflow_architecture.png)

##### Separate DAG processing architecture
In a more complex installation where security and isolation are important, you’ll also see the standalone dag processor component that allows to separate scheduler from accessing DAG files. This is suitable if the deployment focus is on isolation between parsed tasks. While Airflow does not yet support full multi-tenant features, it can be used to make sure that DAG author provided code is never executed in the context of the scheduler.

![dag-processor-architecture.png](https://airflow.apache.org/docs/apache-airflow/stable/_images/diagram_dag_processor_airflow_architecture.png)

#### User interface
Airflow comes with a user interface that lets you see what DAGs and their tasks are doing, trigger runs of DAGs, view logs, and do some limited debugging and resolution of problems with your DAGs.

![dags.png](https://airflow.apache.org/docs/apache-airflow/stable/_images/dags.png)

### DAGs
A DAG (Directed Acyclic Graph) is the core concept of Airflow, collecting Tasks together, organized with dependencies and relationships to say how they should run.

Here’s a basic example DAG:

![basic-dag.png](https://cdn.nlark.com/yuque/0/2024/png/667308/1729960419403-80620a83-428a-44cc-8b30-f928660722f6.png)

It defines four Tasks - A, B, C, and D - and dictates the order in which they have to run, and which tasks depend on what others. It will also say how often to run the DAG - maybe “every 5 minutes starting tomorrow”, or “every day since January 1st, 2020”.

The DAG itself doesn’t care about what is happening inside the tasks; it is merely concerned with how to execute them - the order to run them in, how many times to retry them, if they have timeouts, and so on.

### DAG Runs
A DAG Run is an object representing an instantiation of the DAG in time. Any time the DAG is executed, a DAG Run is created and all tasks inside it are executed. The status of the DAG Run depends on the tasks states. Each DAG Run is run separately from one another, meaning that you can have many runs of a DAG at the same time.

### Tasks
A Task is the basic unit of execution in Airflow. Tasks are arranged into DAGs, and then have upstream and downstream dependencies set between them in order to express the order they should run in.

There are three basic kinds of Task:

- Operators, predefined task templates that you can string together quickly to build most parts of your DAGs.

- Sensors, a special subclass of Operators which are entirely about waiting for an external event to happen.

- A TaskFlow-decorated @task, which is a custom Python function packaged up as a Task.

### Operators
An Operator is conceptually a template for a predefined Task, that you can just define declaratively inside your DAG:

```python
with DAG("my-dag") as dag:
    ping = HttpOperator(endpoint="http://example.com/update/")
    email = EmailOperator(to="admin@example.com", subject="Update complete")

    ping >> email
```

Airflow has a very extensive set of operators available, with some built-in to the core or pre-installed providers. Some popular operators from core include:

- BashOperator - executes a bash command

- PythonOperator - calls an arbitrary Python function

- EmailOperator - sends an email

- Use the @task decorator to execute an arbitrary Python function. It doesn’t support rendering jinja templates passed as arguments.

For a list of all core operators, see: [Core Operators and Hooks Reference](https://airflow.apache.org/docs/apache-airflow/stable/operators-and-hooks-ref.html).

If the operator you need isn’t installed with Airflow by default, you can probably find it as part of our huge set of community [provider packages](https://airflow.apache.org/docs/apache-airflow-providers/index.html). Some popular operators from here include:

- HttpOperator

- MySqlOperator

- PostgresOperator

- MsSqlOperator

- OracleOperator

- JdbcOperator

- DockerOperator

- HiveOperator

- S3FileTransformOperator

- PrestoToMySqlOperator

- SlackAPIOperator

### Sensors
Sensors are a special type of Operator that are designed to do exactly one thing - wait for something to occur. It can be time-based, or waiting for a file, or an external event, but all they do is wait until something happens, and then succeed so their downstream tasks can run.

Because they are primarily idle, Sensors have two different modes of running so you can be a bit more efficient about using them:

- **poke** (default): The Sensor takes up a worker slot for its entire runtime

- **reschedule**: The Sensor takes up a worker slot only when it is checking, and sleeps for a set duration between checks

The **poke** and **reschedule** modes can be configured directly when you instantiate the sensor; generally, the trade-off between them is latency. Something that is checking every second should be in poke mode, while something that is checking every minute should be in **reschedule** mode.

Much like Operators, Airflow has a large set of pre-built Sensors you can use, both in core Airflow as well as via our providers system.

### TaskFlow
New in version 2.0.

If you write most of your DAGs using plain Python code rather than Operators, then the TaskFlow API will make it much easier to author clean DAGs without extra boilerplate, all using the @task decorator.

TaskFlow takes care of moving inputs and outputs between your Tasks using XComs for you, as well as automatically calculating dependencies - when you call a TaskFlow function in your DAG file, rather than executing it, you will get an object representing the XCom for the result (an XComArg), that you can then use as inputs to downstream tasks or operators. For example:

```python
from airflow.decorators import task
from airflow.operators.email import EmailOperator

@task
def get_ip():
    return my_ip_service.get_main_ip()

@task(multiple_outputs=True)
def compose_email(external_ip):
    return {
        'subject':f'Server connected from {external_ip}',
        'body': f'Your server executing Airflow is connected from the external IP {external_ip}<br>'
    }

email_info = compose_email(get_ip())

EmailOperator(
    task_id='send_email_notification',
    to='example@example.com',
    subject=email_info['subject'],
    html_content=email_info['body']
)
```

Here, there are three tasks - **get_ip**, **compose_email**, and **send_email_notification**.

The first two are declared using TaskFlow, and automatically pass the return value of **get_ip** into **compose_email**, not only linking the XCom across, but automatically declaring that **compose_email** is downstream of **get_ip**.

**send_email_notification** is a more traditional Operator, but even it can use the return value of **compose_email** to set its parameters, and again, automatically work out that it must be downstream of **compose_email**.

### Executor
Executors are the mechanism by which task instances get run. They have a common API and are “pluggable”, meaning you can swap executors based on your installation needs.

Executors are set by the **executor** option in the **[core]** section of the configuration file.

Built-in executors are referred to by name, for example:

```ini
[core]
executor = KubernetesExecutor
```

Custom or third-party executors can be configured by providing the module path of the executor python class, for example:

```ini
[core]
executor = my.custom.executor.module.ExecutorClass
```

#### Executor Types
There are two types of executors - those that run tasks locally (inside the **scheduler** process), and those that run their tasks remotely (usually via a pool of workers). Airflow comes configured with the **SequentialExecutor** by default, which is a local executor, and the simplest option for execution. However, the **SequentialExecutor** is not suitable for production since it does not allow for parallel task running and due to that, some Airflow features (e.g. running sensors) will not work properly. You should instead use the **LocalExecutor** for small, single-machine production installations, or one of the remote executors for a multi-machine/cloud installation.

##### Local Executors
Airflow tasks are run locally within the scheduler process.

**Pros**: Very easy to use, fast, very low latency, and few requirements for setup.

**Cons**: Limited in capabilities and shares resources with the Airflow scheduler.

Examples:

- [Local Executor](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/executor/local.html)
- [Sequential Executor](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/executor/sequential.html)

##### Remote Executors
Remote executors can further be divided into two categories:

Queued/Batch Executors

Airflow tasks are sent to a central queue where remote workers pull tasks to execute. Often workers are persistent and run multiple tasks at once.

**Pros**: More robust since you’re decoupling workers from the scheduler process. Workers can be large hosts that can churn through many tasks (often in parallel) which is cost effective. Latency can be relatively low since workers can be provisioned to be running at all times to take tasks immediately from the queue.

**Cons**: Shared workers have the noisy neighbor problem with tasks competing for resources on the shared hosts or competing for how the environment/system is configured. They can also be expensive if your workload is not constant, you may have workers idle, overly scaled in resources, or you have to manage scaling them up and down.

Examples:

- [CeleryExecutor](https://airflow.apache.org/docs/apache-airflow-providers-celery/stable/celery_executor.html)
- [BatchExecutor](https://airflow.apache.org/docs/apache-airflow-providers-amazon/stable/executors/batch-executor.html)

Containerized Executors

Airflow tasks are executed ad hoc inside containers/pods. Each task is isolated in its own containerized environment that is deployed when the Airflow task is queued.

**Pros**: Each Airflow task is isolated to one container so no noisy neighbor problem. The execution environment can be customized for specific tasks (system libs, binaries, dependencies, amount of resources, etc). Cost effective as the workers are only alive for the duration of the task.

**Cons**: There is latency on startup since the container or pod needs to deploy before the task can begin. Can be expensive if you’re running many short/small tasks. No workers to manage however you must manage something like a Kubernetes cluster.

Examples:

- [KubernetesExecutor](https://airflow.apache.org/docs/apache-airflow-providers-cncf-kubernetes/stable/kubernetes_executor.html)
- [EcsExecutor](https://airflow.apache.org/docs/apache-airflow-providers-amazon/stable/executors/ecs-executor.html)

### XComs
XComs (short for “cross-communications”) are a mechanism that let Tasks talk to each other, as by default Tasks are entirely isolated and may be running on entirely different machines.

An XCom is identified by a **key** (essentially its name), as well as the **task_id** and **dag_id** it came from. They can have any serializable value (including objects that are decorated with **@dataclass** or **@attr.define**, see [TaskFlow arguments](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/taskflow.html#concepts-arbitrary-arguments):), but they are only designed for small amounts of data; do not use them to pass around large values, like dataframes.

XComs are explicitly “pushed” and “pulled” to/from their storage using the **xcom_push** and **xcom_pull** methods on Task Instances.

To push a value within a task called “task-1” that will be used by another task:

```python
# pushes data in any_serializable_value into xcom with key "identifier as string"
task_instance.xcom_push(key="identifier as a string", value=any_serializable_value)
```

To pull the value that was pushed in the code above in a different task:

```python
# pulls the xcom variable with key "identifier as string" that was pushed from within task-1
task_instance.xcom_pull(key="identifier as string", task_ids="task-1")
```

Many operators will auto-push their results into an XCom key called **return_value** if the **do_xcom_push** argument is set to **True** (as it is by default), and **@task** functions do this as well. **xcom_pull** defaults to using **return_value** as key if no key is passed to it, meaning it’s possible to write code like this:

```python
# Pulls the return_value XCOM from "pushing_task"
value = task_instance.xcom_pull(task_ids='pushing_task')
```

You can also use XComs in [templates](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/operators.html#concepts-jinja-templating):

```sql
SELECT * FROM {{ task_instance.xcom_pull(task_ids='foo', key='table_name') }}
```

XComs are a relative of Variables, with the main difference being that XComs are per-task-instance and designed for communication within a DAG run, while Variables are global and designed for overall configuration and value sharing.

If you want to push multiple XComs at once or rename the pushed XCom key, you can use set **do_xcom_push** and **multiple_outputs** arguments to **True**, and then return a dictionary of values.

### Variables
Variables are Airflow’s runtime configuration concept - a general key/value store that is global and can be queried from your tasks, and easily set via Airflow’s user interface, or bulk-uploaded as a JSON file.

To use them, just import and call **get** on the Variable model:

```python
from airflow.models import Variable

# Normal call style
foo = Variable.get("foo")

# Auto-deserializes a JSON value
bar = Variable.get("bar", deserialize_json=True)

# Returns the value of default_var (None) if the variable is not set
baz = Variable.get("baz", default_var=None)
```

You can also use them from [templates](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/operators.html#concepts-jinja-templating):

```bash
# Raw value
echo {{ var.value.<variable_name> }}

# Auto-deserialize JSON value
echo {{ var.json.<variable_name> }}
```

Variables are global, and should only be used for overall configuration that covers the entire installation; to pass data from one Task/Operator to another, you should use XComs instead.

We also recommend that you try to keep most of your settings and configuration in your DAG files, so it can be versioned using source control; Variables are really only for values that are truly runtime-dependent.

For more information on setting and managing variables, see [Managing Variables](https://airflow.apache.org/docs/apache-airflow/stable/howto/variable.html).

### Params
Params enable you to provide runtime configuration to tasks. You can configure default Params in your DAG code and supply additional Params, or overwrite Param values, at runtime when you trigger a DAG. Param values are validated with JSON Schema. For scheduled DAG runs, default Param values are used.

Also defined Params are used to render a nice UI when triggering manually. When you trigger a DAG manually, you can modify its Params before the dagrun starts. If the user-supplied values don’t pass validation, Airflow shows a warning instead of creating the dagrun.
