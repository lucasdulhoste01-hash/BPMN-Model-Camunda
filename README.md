# Credit Application Process — BPMN on Camunda 8

A BPMN model of a bank's credit approval process, built and executable on Camunda 8. A customer submits a credit request, the bank verifies the file, a scoring system grades it, and the process ends with an approval or a rejection, always communicated to the customer.

> Artificial intelligence and Business intelligence / Lucas Dulhoste, Clément Henaff, Rayan Dellas , 2026/2027

## Resources

- Oral presentation slides: [docs/oral-presentation.pdf](docs/oral-presentation.pdf)
- Textual description of the process: [docs/Credit_Application_Process.pdf](docs/Credit_Application_Process.pdf)

## Table of contents

- [Why this project](#why-this-project)
- [Overview](#overview)
- [Model](#model)
- [Process in detail](#process-in-detail)
- [Scoring system](#scoring-system)
- [Forms](#forms)
- [Known limitations](#known-limitations)
- [How to run it](#how-to-run-it)
- [Team](#team)

## Why this project

We chose bank credit approval because it is a highly verified and regulated process, which makes it a good fit for BPMN. Before lending money, a bank must check the customer's identity, documents and financial situation, then the project itself. These checks follow precise rules, with clear decisions at each stage, and the bank must be able to justify them.

The process also mixes human tasks (the advisor) and automated tasks (calculations, notifications), parallel branches, early rejections, and a scoring system. So the main BPMN elements all have a real use here.

## Overview

The decision is not made "by feeling". The file receives a **score**, and the decision follows from it:

- **Overall score ≥ 70**: credit automatically approved
- **50 ≤ score < 70**: manual review by a banking advisor
- **Score < 50**: credit automatically rejected

The system decides the clear cases, and a human keeps control of the ambiguous ones. The process ends with the decision and its communication to the customer. Contract signing and fund disbursement are out of scope.



## Model

Full BPMN model:

![Full BPMN model](images/Full BPMN model.png)

Part 1 — Verification of the customer's file:

![Part 1 of the BPMN model](images/Part 1 of the BPMN model.png)

Part 2 — Project evaluation and decision:

![Part 2 of the BPMN model](images/Part 2 of the BPMN model.png)

## Process in detail

### Part 1 — Verification of the customer's file

The process begins at the "Credit Application" start event. A parallel gateway immediately starts two branches that run at the same time:

- **Customer branch**: the customer completes the credit application form.
- **Administrative branch**: the bank records the request and opens the case, receives the supporting documents, checks the completeness of the application, archives the documents, then verifies the customer's identity, age and possible banking ban.

A file that fails a check is rejected and the customer is notified. A file that passes both checks waits at a second parallel gateway until the customer's form is also complete. Only then does project evaluation begin.

| Step | Task | Description |
|---|---|---|
| 1 | Credit Application (start event) | Starts the process and launches both branches through a parallel gateway. |
| 2 | Fulfill the form by the client (user task) | The customer enters the amount, income, savings, term, project description and employment status. All fields are mandatory. |
| 3 | Record the request and open the case, Receive the supporting documents | Registration of the request and collection of the evidence needed for review. |
| 4 | Verify the completeness of the application (script task) | Produces CONFORME or NON CONFORME. A non-compliant file leads to a rejection notification and the "Rejection of the application" end event. |
| 5 | Archive the documents, Verify the customer's identity, age and possible banking ban | Produces ELIGIBLE or NON ELIGIBLE. A non-eligible customer is notified and the branch ends. |
| 6 | Parallel gateway (join) | Waits for both branches. Project evaluation starts only when the customer form is complete and both checks are passed. |

### Part 2 — Project evaluation and decision

| Step | Task | Description |
|---|---|---|
| 7 | Verify the project (user task) | A banking advisor approves or rejects the project, gives it a grade from 1 to 10, and writes a justification. A rejected project goes to the rejection notification. |
| 8 | Calculate the financial indicators (3 script tasks) | Savings coverage, monthly payment, debt-to-income ratio. |
| 9 | Obstructing criterion? (gateway) | Rejection if the debt-to-income ratio is above 50% or the savings coverage is below 10%. |
| 10 | Calculate the consistency score | Points from the two indicators, with a bonus for a very low debt ratio. Maximum 100. |
| 11 | Calculate the overall score | Weighted combination of the consistency score and the advisor's grade. |
| 12 | Automatic decision (gateway) | Automatic rejection, manual review, or automatic approval depending on thresholds. |
| 13 | Manually analyze the file (user task) | For intermediate scores, the advisor grants or rejects the credit and must write a justification. |
| 14 | Notify the customer and finish | The customer is notified of the agreement or the rejection, then the process ends. |

## Scoring system

### Financial indicators

| Indicator | Formula |
|---|---|
| Savings coverage (%) | (Savings ÷ Amount requested) × 100 |
| Monthly payment (€) | Amount requested ÷ Loan term (months) |
| Debt-to-income ratio (%) | (Monthly payment ÷ Monthly income) × 100 |

### Consistency score

`scoreCoherence` = debt-ratio points + savings-coverage points + low-debt bonus (maximum 100).

| Debt-to-income ratio | Points | Savings coverage | Points |
|---|---|---|---|
| ≤ 30% | 50 | ≥ 75% | 40 |
| > 30% to 35% | 40 | 50% to < 75% | 30 |
| > 35% to 40% | 25 | 25% to < 50% | 20 |
| > 40% to 50% | 10 | 10% to < 25% | 10 |
| > 50% | 0 | < 10% | 0 |

A bonus of **10 points** is added when the debt-to-income ratio is at or below 10%.

### Overall score

```
Overall score = (Consistency score × 0.70) + (Project grade × 10 × 0.30)
```

Example: a consistency score of 80 and a project grade of 7 give (80 × 0.70) + (7 × 10 × 0.30) = **77**, so the credit is automatically approved.

### Decision thresholds

| Overall score | Outcome |
|---|---|
| Below 50 | Automatic rejection |
| 50 to below 70 | Manual credit-file review |
| 70 or higher | Automatic approval |

## Forms

| Form | Used by | Content |
|---|---|---|
| **Credit application** | Customer | Requested amount, monthly income, total savings, loan term in months (minimum 60), project description, employment status (employee or unemployed). All fields are mandatory. |
| **Project analysis** | Banking advisor | Customer data, decision (approved or rejected), project grade from 1 to 10, written justification. |
| **Credit File Analysis** | Banking advisor | Consistency score, project grade, debt-to-income ratio, overall score, amount, income, monthly payment, decision (grant or reject), mandatory written justification. |

## Known limitations

This is a simplified model:

- **The completeness check and the eligibility check use random results.** They do not really check documents, identity or banking bans.
- The monthly payment is amount ÷ term. It does not include interest, insurance or fees.
- The debt ratio does not include the customer's existing monthly debts.
- The BPMN does not list the specific supporting documents that are required.
- The process ends with the decision and its notification. There is no contract signing or fund disbursement.

## How to run it

This project targets **Camunda 8**.

### Option A — Camunda 8 SaaS (Web Modeler)

1. Create an account on Camunda 8 SaaS and create a cluster.
2. In Web Modeler, create a project and import `bpmn/credit-process.bpmn` and the three forms from `forms/`.
3. Check that each user task is linked to its form:
   - "Fulfill of the form by the client" → Credit application
   - "Verify the project" → Project analysis
   - "Manually analyze the file" → Credit File Analysis
4. Deploy the diagram to your cluster.
5. Start a new instance.

### Option B — Camunda Desktop Modeler

1. Install Camunda Desktop Modeler and open `bpmn/credit-process.bpmn`. The forms must be available alongside it.
2. Deploy the process to a Camunda 8 cluster (SaaS or local) with the Deploy button.

### Using the process

1. Open **Tasklist** and complete the customer form with a test case: amount, income, savings, a term of at least 60 months, a project description and an employment status.
2. As the advisor, complete "Verify the project" with a decision, a grade from 1 to 10 and a justification.
3. Follow the instance in **Operate** to see which path it takes.

Because the completeness and eligibility checks are random, run several instances to see the different outcomes.


## Team

- Clément Henaff
- Lucas Dulhoste
- Rayan Dellas
