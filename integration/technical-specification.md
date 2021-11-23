# Technical Specification

## **Technology**

List every top-level framework, SDK, and toolkit used in the project. Version need to be specified.

| Framework/SDK/Toolkit | **Version** | **Description** |
| --------------------- | ----------- | --------------- |
| Lavarel               | 5.4         | PHP framework   |
| ...                   | ...         | ...             |

## **Infrastructure**

List every server, app and database instances including access url and credentials.

| **Type**          | **Name**                      | **Details**                                         |
| ----------------- | ----------------------------- | --------------------------------------------------- |
| **EC2 Server**    | ...** **(instance identifier) | Type, IP, DNS Name, Security Group, User, Pass, Key |
| **Database**      | ...                           | Host, User, Password, Root Password                 |
| **Connector App** | ...                           | Directory, URL                                      |
| **DB Management** | ...                           | Directory, URL, Auth User, Auth Pass                |
| **Media Uploads** | ...                           | Directory, URL                                      |

## **Architecture Diagram**

Illustrate how instances and middlewares setup and communicate with each other.

## **Sequence Diagram**

Illustrate how data goes from one platform to the other. Break down whole integration solution to every single task, which has it's own start point, end point, actions, flow direction and engaged entities. Solution of tasks are extracted from business workflow, which differ from one project to another.

## Field Mapping

Mapping fields between two tender platforms, normally one of them will be Shopify. Typically, customer, product, variant, stock, order, line item, fulfillment, and refund will be mandatory objects.

|                     | _**Shopify**_ | _**The Other Platform**_ |                |
| ------------------- | ------------- | ------------------------ | -------------- |
| **Product**         | **Field**     | **Field**                | **Notes**      |
| Object Level        | product       | ...                      |                |
|                     | id            |                          | auto-increment |
| metafields          | key           | ...                      |                |
|                     | value         | ...                      |                |
|                     | value\_type   | string                   |                |
|                     | namespace     | ...                      |                |
|                     | description   | ...                      |                |
|                     |               |                          |                |
| **Product Variant** | **Field**     | **Field**                | **Notes**      |
| Object Level        | variant       | ...                      |                |
|                     | id            |                          | auto-increment |
