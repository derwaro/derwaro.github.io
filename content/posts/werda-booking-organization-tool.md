---
title: "Werda - Booking Organization Tool"
date: 2023-05-03T22:27:47-06:00
lastmod: 
draft: true
tags: ["flask", "api",]
categories: ["showcase"]
---
# Content

description of content of showcase

# Background or How I Got To Where I Got

In 2017 I changed from a head on, face-to-face position to a more administrative, back-office-y position. My supervisors proposed me to change into a coordinator position for a team of in-house interpreters. I quickly accepted since it sounded like a challenging position and furthermore after years of directly working on a day to day basis with persons in difficult, uncertain situations it relieved me of growing pessimism (or even second hand trauma).

The team of interpreters was split into two teams, each roughly making up half of the workload. One team was made up of the Arabic and Somali, the other of Farsi, Afghani Languages (Dari, Pashtu, Urdu,...) and Russian. I was assigned to the later team and instructed by my predecessor. During this introduction I already sensed some pain points in the day to day workflows and also recurring tasks (especially the monthly billing). Since I just started out I first of all tried to grasp the whole picture to be able to think and plan improvements and measures.

# Initial Situation

Working as an interpreter is a classic self-dependent work in Austria, one works independently for different clients. Still, since my employer worked with migrants coming to Austria there was a fixed need for a certain amount of interpreters at most of the times. They decided to offer some, later all, interpreters a position in a newly created team. The idea was that this teams not only covers the need for interpretation in-house, but also for other sites/offices within the same company and furthermore for external clients. The need in-house would see coverage for most of the languages during office hours (interpreters on call, in person). External clients (both within the same company and from other companies) would be covered through reservation on a per booking basis (with fixed hourly rates).  
For me as a coordinator this meant to organize my team in a way that enough interpreters are available for colleagues in-house and to organize external appointments in such a manner that the most of them could be covered by our interpreters.

## In-House Organization
The planning was (and more or less still is) done on a weekly basis. When I started out the weekly plan was done in an Excel Sheet. The Sheet resided in an `.xls` file, which contained one sheet for each week of the year. Each sheet contained 5 columns for the weekdays from Monday to Friday. Each day was split up into Morning (8-12 o'clock) and Afternoon (12-16 o'clock). Inside the section for the morning the top was deserved for the Farsi/Dari Team and the bottom for the Arabic team. Inbetween we had to fit Russian, Somali and various "smaller" languages (for example, Bangla). Where each language was located was implicit knowledge for us and more importantly for the in-house colleagues. Meaning new colleagues had to learn who speaks what languages by trial and error, asking their experienced colleagues or by consulting in a list with names and languages we sent out, when there were changes in our team.

In the file we manually copied, that is added, the next week manually and then on the basis of the current week worked out the plan for the next week. That meant we had to check who was missing in the current week due to illness or vacation and who had differing working hours in the current week. Since we deleted rows for those who were missing, we manually had to readd those rows and again enter the working hours for that person.

Since an interpreter may be available for the whole day, but have to assist an external appointment during certain hours we added their absent times. The problem was, when there were many absent times and they didn't fit in their respective column.
Furthermore each interpreter was available by phone (the office was split up between 3 floors) and their respective direct number was in that sheet too.

As already mentioned we planned on a weekly basis, doing the biggest load of work from Thursday Afternoon until Friday Noon. At Noon we created `.pdf` files for in-house colleagues and the interpreters and sent them out via mail. Due to how the Excel was set up there was a lot of room for errors: some interpreter may have notified us or our supervisor that he/she will be missing due to personal reasons at a certain time or day of the next week. We only received this feedback after sending the plan out to the interpreter team. So then we had to replan and send out a new `.pdf` to them. Obviously we were worse of, sending out the plan to our in-house colleagues and only after receive a notification from an interpreter. Then we had to send out confusing mails to the colleagues and to the interpreter team.

I mocked up a quick sketch of those plans (thanks to [tablesgenerator.com](https://www.tablesgenerator.com/markdown_tables#)):
(be sure to horizontal scroll this beauty)
```
|      	| Monday    	|                         	|     	|     	| Tuesday 	|           	|     	|     	| Wednesday 	|               	|     	|     	| Thursday  	|                         	|     	|     	| Friday    	|                        	|     	|     	|
|------	|-----------	|-------------------------	|-----	|-----	|---------	|-----------	|-----	|-----	|-----------	|---------------	|-----	|-----	|-----------	|-------------------------	|-----	|-----	|-----------	|------------------------	|-----	|-----	|
|      	|           	| details                 	| EXT 	|     	|         	| details   	| EXT 	|     	|           	| details       	| EXT 	|     	|           	| details                 	| EXT 	|     	|           	| details                	| EXT 	|     	|
| M    	| Jorge     	| absent 9-12             	| 123 	|     	| Kim     	| here 8-11 	| 124 	|     	| Jorge     	|               	| 123 	|     	| Sheila    	|                         	| 123 	|     	| Jorge     	|                        	| 123 	|     	|
| O    	| Kim       	|                         	| 124 	|     	| Carter  	|           	| 126 	|     	| Kim       	|               	| 124 	|     	| Carter    	| absent 8-9              	| 126 	|     	| Kim       	| here 8-11              	| 124 	|     	|
| R    	| Loretta   	| here 11-12              	| 125 	|     	| Max     	|           	| 125 	|     	| Loretta   	| here 8-10     	| 125 	|     	| Dave      	|                         	| 127 	|     	| Loretta   	|                        	| 125 	|     	|
| N    	| Carter    	|                         	| 126 	|     	|         	|           	|     	|     	| Carter    	| absent 10-11  	| 126 	|     	|           	|                         	|     	|     	|           	|                        	|     	|     	|
| I    	| Dave      	| absent 8-9              	| 127 	|     	|         	|           	|     	|     	|           	|               	|     	|     	|           	|                         	|     	|     	|           	|                        	|     	|     	|
| N    	|           	|                         	|     	|     	|         	|           	|     	|     	|           	|               	|     	|     	|           	|                         	|     	|     	|           	|                        	|     	|     	|
| G    	| Kyle      	|                         	| 221 	|     	| Bryan   	|           	| 222 	|     	| Kyle      	|               	| 221 	|     	| Bryan     	|                         	| 222 	|     	| Kyle      	|                        	| 221 	|     	|
| M    	| Bryan     	| absent 8-9.30, 11.30-13 	| 222 	|     	| Herman  	|           	| 223 	|     	| Herman    	| here 11-12    	| 223 	|     	| Katherine 	| absent 8-8.30, 11.30-13 	| 224 	|     	| Bryan     	|                        	| 222 	|     	|
| O    	| Herman    	|                         	| 223 	|     	|         	|           	|     	|     	| Katherine 	|               	| 224 	|     	| Marion    	|                         	| 225 	|     	| Herman    	| absent 8-9.30          	| 223 	|     	|
| R    	| Katherine 	|                         	| 224 	|     	|         	|           	|     	|     	|           	|               	|     	|     	| Andre     	|                         	| 221 	|     	| Katherine 	|                        	| 224 	|     	|
| N... 	|           	|                         	|     	|     	|         	|           	|     	|     	|           	|               	|     	|     	|           	|                         	|     	|     	| Marion    	| here 10.30-12          	| 225 	|     	|
| ~~~~ 	| ~~~~      	| ~~~                     	| ~~~ 	| ~~~ 	| ~~~     	| ~~~       	| ~~~ 	| ~~~ 	| ~~~       	| ~~~           	| ~~~ 	| ~~~ 	| ~~~       	| ~~~                     	| ~~~ 	| ~~~ 	| ~~~       	| ~~~                    	| ~~~ 	| ~~~ 	|
| A    	| Jorge     	|                         	| 123 	|     	| Kim     	|           	| 124 	|     	| Jorge     	|               	| 123 	|     	| Sheila    	|                         	| 123 	|     	| Jorge     	| absent 12-16           	| 123 	|     	|
| F    	| Kim       	| absent 14-16            	| 124 	|     	| Carter  	|           	| 126 	|     	| Kim       	| here 13-15.30 	| 124 	|     	| Carter    	|                         	| 126 	|     	| Kim       	|                        	| 124 	|     	|
| T    	| Loretta   	|                         	| 125 	|     	| Max     	|           	| 125 	|     	| Loretta   	|               	| 125 	|     	| Dave      	|                         	| 127 	|     	| Loretta   	|                        	| 125 	|     	|
| E    	| Carter    	| here 12-15              	| 126 	|     	|         	|           	|     	|     	| Carter    	|               	| 126 	|     	|           	|                         	|     	|     	|           	|                        	|     	|     	|
| R    	| Dave      	|                         	| 127 	|     	|         	|           	|     	|     	|           	|               	|     	|     	|           	|                         	|     	|     	|           	|                        	|     	|     	|
| N    	|           	|                         	|     	|     	|         	|           	|     	|     	|           	|               	|     	|     	|           	|                         	|     	|     	|           	|                        	|     	|     	|
| O    	| Kyle      	|                         	|     	|     	| Bryan   	|           	| 222 	|     	| Kyle      	|               	| 221 	|     	| Bryan     	|                         	| 222 	|     	| Kyle      	|                        	| 221 	|     	|
| O    	| Bryan     	| absent 12-13            	|     	|     	| Herman  	|           	| 223 	|     	| Herman    	| here 15-16    	| 223 	|     	| Katherine 	|                         	| 224 	|     	| Bryan     	|                        	| 222 	|     	|
| N    	| Herman    	|                         	|     	|     	|         	|           	|     	|     	| Katherine 	|               	| 224 	|     	| Marion    	| absent 12-13.30         	| 225 	|     	| Herman    	| absent 14-15, 15.30-16 	| 223 	|     	|
| A    	| Katherine 	|                         	|     	|     	|         	|           	|     	|     	|           	|               	|     	|     	| Andre     	|                         	| 221 	|     	| Katherine 	|                        	| 224 	|     	|
| ...  	|           	|                         	|     	|     	|         	|           	|     	|     	|           	|               	|     	|     	|           	|                         	|     	|     	| Marion    	|                        	| 225 	|     	|
|      	|           	|                         	|     	|     	|         	|           	|     	|     	|           	|               	|     	|     	|           	|                         	|     	|     	|           	|                        	|     	|     	|
|      	|           	|                         	|     	|     	|         	|           	|     	|     	|           	|               	|     	|     	|           	|                         	|     	|     	|           	|                        	|     	|     	|
|      	|           	|                         	|     	|     	|         	|           	|     	|     	|           	|               	|     	|     	|           	|                         	|     	|     	|           	|                        	|     	|     	|
|      	|           	|                         	|     	|     	|         	|           	|     	|     	|           	|               	|     	|     	|           	|                         	|     	|     	|           	|                        	|     	|     	|
```

This is obviously not the real thing (there is still information missing. We had color codes for the teams and each interpreter had his/her main language in brackets attached to his/her name,...), but enough to see yet another pain point: The plan is quite extensive. There are a lot of lines, a lot of columns for each line, day, interpreter. To reduce the size we removed rows where possible: If for example in a given week there were at most only 5 instead of 7 interpreters in a team, we removed two rows.

### Pain Points In-House Organization
Let's sum up possible pain points for us coordinators, the interpreters, and colleagues:
* No direct visibility of spoken languages for each interpreter: Some interpreters spoke up to 5 different languages, but were represented with their "main" language only.
* The file was mostly only editable by one coordinator at a time. Since moving rows or similar moved around a whole lot of stuff.
* Correcting plans due to errors in the planning process involved sending out new `.pdf`, resulting in unnecessary inbox noise or plain ignoring from colleagues.
* Rigid format: Using a `.pdf` limited us in our possibilities to update the plan during the week. For example: Jorge falls ill on Tuesday and will not come to work for the rest of the week. This involved in the best of the cases an e-mail to all teams explaining that he will be missing and hope that everyone sees and processes that mail. This get's even messier, when the intial heads-up from the sick interpreter is only for one day and is prolonged afterwards. Since we maybe would get away with a simple mail for one day, but would need a new `.pdf` if it affects various days. Furthermore: Keep in mind that the absences are external appointments. We should cover as many of them as possible. So appointments initially planned for the now sick interpreter would have to be covered by other interpreter, which in turn mangles up the plan even more.
* There were occasions, when the mobile landline phones didn't work (mostly the batteries fault). We would then have to send out an e-mail notifying colleagues of a temporary substitute extension for that interpreter. 
* External appointments that were sent late (that is after the formal deadline Thursday Afternoon in the week before the appointment), were not represented in the plan and caused confusion when an interpreter that should be available suddenly is not.
* We had no way to be sure that everybody is fulfilling his/her workload, the hours in their contract. The table did not include exact hours and if someone as exception stayed longer or shorter we had no way to pass that information into the next week (except error prone notes or e-mails to ourselves). This resulted in some cases where interpreters had a lot of extra hours or were even missing hours.
* Break times: We later had to include a column with the break time for each interpreter, since we had to make sure that there were always enough of them available for colleagues.

Those were the most crucial points we had to struggle with. So far so good, let's get into the details for the external appointments.

## External Appointments
When I joined the team back in 2017, the new hot way to book an interpreter for an external appointment was through a form on Google Forms. Before that bookings arrived through a form in a `.doc` file (horrible to manage. Languages were hard corded and if something changed we had to send out the new version per e-mail to our clients). Naturally Google Forms was already a big improvement for clients and coordinators.
Appointments were copied from a linked Google Sheet to a local `.xlxs` (later `.xlxm`) file. We planed and organized the interpreters in it filtering for week, day, status, cancellation status, and team (based on languages). At the end of the week we created two separate `.pdf` files, which were sent out to the interpreters. To confirm appointments with clients we had to do two separate procedures since "clients" were split up into "internal" and "external". Internal meant offices from within our company, but not located in our building. For them we created a big `.pdf` file containing all future appointments and saved it onto a network drive accessible by all offices/buildings. External refers to all other clients (mostly other NGOs, hospitals and alike). For them we sent out individual e-mails confirmations by hand(!). The workflow for this was to filter out each client copy the appointments into an e-mail and send it, resulting in about 30 to 40 tediously created and sent e-mails.

Compared to the form-in-a-doc-file the Google Form was a big improvement and generally received well by our clients. However we initially had a lot of repeating but varied questions. For example:
* Double contact details: We asked for a contact responsible for the appointment and another contact responsible for the or holding the appointment.
* Repetitive information: We asked internal clients for their complete billing address and cost center.

Over time I analyzed which details were redundant and continuously improved the form reducing it by 1 page. To pick up those examples from above:
* Double contact details: Since the most part of appointments was booked and held by the same person, I simply asked for the e-mail contact for the responsible (think: user e-mail or "unique" identifier) and for way to contact the person holding the appointment (usually we received at least a name a phone number through this). This normally resulted in complete contact details: e-mail, phone number, name. 
* Repetitive information: We stopped asking internal clients for their complete billing details and instead asked them for their cost center number instead. Since we had an complete internal directory we were able to deduce the billing address from the cost center number.



### Pain Points External Appointments

## Billing

### Pain Points Billing

## In Search Of An Improvement

## Werda

### In-House Views

### External Appointment Views

### Interpreter Views