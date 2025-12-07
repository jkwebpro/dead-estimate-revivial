## Project Name:  
_Dead Estimate Revival_

## Stakeholders
1. [Customer: Sales Manager/Person]
2. [Project Architect]
3. [Salesperson]

## Project Context
For each stakeholder capture a problem statement and their user story:
1. [Customer: Sales Manager/Person]
	1. **Problem statement**: Estimates issued but prospects ghost.
	2. **User story**: As a [Customer: Sales Manager/Person], I want to have an automated customer follow-up system, so that I can: revive cold leads, shorten the sales cycle, and ensure consistency. 
2. [Project Architect]
	1. **User story**: 
		1. As a [Project Architect], I want to have a simple automation that is reusable, so that it is quick and easy to deploy, it should be modular. 
		2. As a [Project Architect], I want to have an automation that can handle different trigger and alerting scenarios easily (SMS and email, voice ready for bonus points), so that we don’t have to change the project for small customer use case variances. 
3.  [Salesperson]
	1. **User story**: As a [Salesperson], I want an automation that I can demo easily (like on the fly), so that I can easily show the value and flexibility of the automation and hopefully achieve that “wow” factor at demo time. 

Customer refers to the person receiving a quote, in this situation. 
Owner refers to the person who owns the automation workflow. 
Project architect and salesperson refer to my company, as the team responsible for delivering this product. 

## Description / Background:
- [Customer: Sales Manager/Person] : 
	- I send out 20 quotes a week, follow up once, but get busy and forget to stick with it. The customer goes silent.
	- I have some customers who prefer SMS over email, vice-versa, and some who like both.
	- I have some customers who are not open to text follow-up methods and prefer voice (phone calls). 

## Deliverables:
Capture the deliverable(s): 
1. Owner
	1. n8n automation workflow with the following requirements:
		1. Customer to quote data storage method
		2. Customer communication methods:
			1. SMS
			2. Email
2. Project Engineer
	1. Standard project documentation

## Constraints:
1. Deadlines: none for demo
2. General:
	1. Workflow needs to be flexible for input trigger methods (email, SMS, or updated spreadsheet)
	2. Workflow must be capable of doing SMS and email communications in a both/and and either/or configuration 
	3. Workflow must have a mechanism to bypass, or opt-out of the automated communications
3. Platforms (suggested):
	1. Customer (recipient) data: Google Sheets 
	2. Customer (recipient) SMS: Twilio? Is there a path with a google workspace product?
	3. Customer (recipient) email: Gmail
4. Budget:  TBD

## Inputs Provided:
> documents, links, notes for additional context/training

## Evaluation Criteria:
- Ability to identify, and resolve, all necessary decision points before building 
- Ability to resolve all user stories
- Ability to comply with constraints
- Accuracy  
- Real-world application and utility 

## Reference Examples:
This is the first ever implementation of this effort. This will serve a demo workflow that hopefully serves as a good template for owners. 

Follow-up message 1:

Follow-up message 2: