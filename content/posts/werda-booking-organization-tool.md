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
Summing up, these were some of the problematic things related to external appointments for all those involved:
* Lengthy booking form with no way to store repetitive information such as contact information or location
* No way to directly identify a booked appointment through an id or unique identifier: when external clients called us or emailed us regarding an appointment, if was necessary to ask them or for them to provide us with a lot of information to correctly identify the correct appointment: date, hour, required language,...
* Dependence on Google Forms and Sheets: All information was accessible for Google. Although the link to the form was public, luckily there was never any abuse, but also no way for us to make sure, that an appointment was really booked and required by a specific person, since there were no logins or similar. 
* Error prone "import" process: Appointments were booked through a Google Form, which stored it's results in a Google Sheet. Since we relied on various filters and needed statistics and reports, we manually copied new appointments to a local Excel file for further processing.
* No direct feedback for clients if appointment is confirmed: For internal clients, we stored a pdf overview of the upcoming appointments on a internally public network drive on a daily basis (I automated this through an Excel Macro). For external clients we manually sent out emails to each one of them on a weekly basis and if short term appointments were made we sent out special email confirming those, again manually. The general understanding was that an appointment is confirmed unless we coordinators contact the booker.
* No same time editing: I would say our Excel file was not even that complicated (I've seen more complex ones), but still for some reason I never managed to get the same time editing working. Maybe the network was the culprit or we did something wrong, but there were always duplicated created on the personal user folder or unsaved changed and so on. This resulted effectively in us coordinators blocking each other.
* Typos in cost center numbers/identifiers: Since every client could enter whichever cost center number he/she liked to, there often happened typos, which had to be cleaned up in the billing process.

## Billing
We billed our clients monthly based on their actual appointments, whereas during coordination we already made sure that certain minimum times were enforced: min 1 hour duration for personal appointments, min 30 minutes for phone and videocall. From there it was necessary to clean up the data: were there no empty appointments, as in is there a interpreter assigned to each appointment? Are all cost centers correct and existing in the Excel sheet which generates the bills?

### Pain Points Billing
* Typos in cost centers identifiers: Cost centers are identified by a 4 to 6 digit number. The first 4 digits describe the main cost center and the additional 2 digits describe a dependance or subdivision of it. However it is also possible to denominate a main cost center with 6 digits by appending two 0s to it: that way 1234 and 123400 were the same main cost center, but for Excel this was not seen as the same number
* Clients could enter whatever cost center they liked or thought up: There was no way to validate those digits through Google Forms.
* External clients had no official cost center number: I assigned fictional cost center numbers, in that case more client numbers, to clients external to our company. However officially they didn't know their number, although it was present on the bills. So for external clients it was necessary to enter "0000" as cost center in the Google Form and I had to substitute that with the fictional number during the clean up process. 

There were some more difficulties with billing[^1], but I have not yet implemented a complete billing process on my platform, so I won't treat them here. 

# In Search Of An Improvement
I spent quite some time during off hours in the coordination business searching for pre-existing solutions I could use out of the box or adapt to our necessities and requirements. Just some off the top of my head, that I still remember:
* Microsoft Office Access - While in Highschool I learned to use it, but still it was more of a struggle than anything else. It did, if only, adapt to our in house needs, but not to our external clients. Reports were nice, but still, couldn't for the life of me, get it to serve our even most basic needs.
* [Easy!Appointments](https://easyappointments.org/): My understanding of PHP is limited so I couldn't accommodate this tool to ours business needs. Easy!Appointments does not provide a way to let the client define the duration of the appointment, which was crucial to our needs. Furthermore it is not possible for the client to provide a location for the appointment, which, also, was absolutely necessary for our clients. 
* Various other apps, plugins and such for frameworks or CMS: The difficulty there was that I couldn't find a service that provided everything needed: Various definable services (languages) with modalities (in-sitio, telephone, videocall), at client provided locations with client provided length. All of that with the possibility to filter, coordinate and assign those appointments and share appointments as well with clients as with interpreters.

Actually the last point sums up pretty well the bare minimum requirements to a solution. I'll get to them again later on. For now, I'll conclude this part, with the revelation that every pre-existing solution would require too much customization and therefore either forks or rewrites of large parts, which mean a lot of in-depth digging into their code, that a custom solution is more than justified. 

# Werda


## In-House Views
For our in-house views I first of all replicated the existing style, that is: a big clunky site that looks exactly like the exported Excel sheet looked like in it's pdf incarnation. However already here there were many solutions possible and implemented by me. First of all this view replicated the former pdf table style as I mentioned. So interpreters are grouped into the three main language groups. The view however, by means of a button, allowed to switch between this *grouped* style **and** a *detailed* view. This *detailed* view, may have been a bit of an overkill, but for those wanting to have a holistic view of the whole week in a very detailed view it offered: Each day being a column and in each column a detailed section for each and every language covered by our team. The resulting table obviously is huge, but you get *everything* from it.
A further improvement, which applied to all views, was to make the phone numbers for the interpreters clickable. The company did not offer a way to call phones from desktops, but since work from home was very prevalent due to some world wide pandemic, many were working from home and outfitted with company mobile phones. Therefore being able to simply click-to-call the numbers was a big improvement over copying or remembering/replicating the phone numbers. Also: if for some reason an interpreter had to change their phone number, the coordinators are able to update the entries in the backend and have it propagated to all colleagues automatically. 

## External Appointment Views

## Interpreter Views

[^1]: Bills effectively were created through the merge letter functionality of Word, taking an Excel sheet as the basis/input. While more or less efficient there were many edge cases that did not fit nicely into that merge letter and if you've worked with it's conditional loops you know how strange Word behaves in that case. Furthermore, for our internal book keeping the bills file name had to match a certain pattern based on cost center numbers involved and the continous bill number. I solved that writing a bash script using less, awk, sed and pdftk. And from there the bills had to be sent one by one to the accounting system: I solved that by writing a small VBA script that took a file list created by the bash script as input and sent each email through Outlook.