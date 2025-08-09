# Setting the scene
We're Given the following text to set the mood:

```txt
A body was found floating near the docks of Coral Bay Marina in the early hours of August 14, 1986. Your job detective is to find the murderer and bring them to justice. This case might require the use of JOINs, wildcard searches, and logical deduction. Get to work, detective.
```

## Initial information
- Location: Coral Bay Marina
- Type: Murder
- date: 19860814 (dates are in yyyymmdd format)

## The Databases

- crime_scene:
  
  | Column      | Type    |
  | ----------- | ------- |
  | id          | INTEGER |
  | date        | INTEGER |
  | location    | TEXT    |
  | description | TEXT    |

- person:
  
  |Column|Type|
  |---|---|
  |id|INTEGER|
  |name|TEXT|
  |alias|TEXT|
  |occupation|TEXT|
  |address|TEXT|

- interviews:
  
  |Column|Type|
  |---|---|
  |id|INTEGER|
  |person_id|INTEGER|
  |transcript|TEXT|

- hotel_checkins:
  
  |Column|Type|
  |---|---|
  |id|INTEGER||
  |person_id|INTEGER|
  |hotel_name|TEXT|
  |check_in_date|INTEGER|

- surveillance_records:
  
  |Column|Type|
  |---|---|
  |id|INTEGER|
  |person_id|INTEGER|
  |hotel_checkin_id|INTEGER|
  |suspicious_activity|TEXT|
  
- confessions:
  
  |Column|Type|
  |---|---|
  |id|INTEGER|
  |person_id|INTEGER|
  |confession|TEXT|

# The Investigation

## The Crime Scene
Let's start by investigating the crime scene as a whole;

```sql
select * from crime_scene where location = "Coral Bay Marina"
```

|id|date|location|description|
|---|---|---|---|
|43|19860814|Coral Bay Marina|The body of an unidentified man was found near the docks. Two people were seen nearby: one who lives on 300ish "Ocean Drive" and another whose first name ends with "ul" and his last name ends with "ez".|
### Intel
- witness1 address: 300ish "Ocean Drive"
- witness2 name: first name ends in "ul" and last name "ez"

## Digging into the details
SQL allows us to do fuzzy matching using the wildcard character `%`;

```sql
select * from person where address like "3% Ocean Drive" or name like "%ul %ez"
```

|id|name|alias|occupation|address|
|---|---|---|---|---|
|5|Michael Santos|Silent Mike|Bartender|33 Ocean Drive|
|101|Carlos Mendez|Los Ojos|Fisherman|369 Ocean Drive|
|102|Raul Gutierrez|The Cobra|Nightclub Owner|45 Sunset Ave|
|105|Victor Martinez|Slick Vic|Bartender|33 Ocean Drive|
Let's dissect this information to narrow it down!; according to our information Raul Gutierrez, id 102, is the only person that matches the First name "ul" last name "ez". So, that's important to take note. Now in order to figure out witness1 we see that information says "300ish" Ocean drive, and the only one that matches the 300ish range is Carlos Mendez, id 101.

## Diving Deeper

With our list of 101,102 (Carlos and Raul), we can do a little bit of diving. 

```sql
select transcript from interviews where person_id in (101,102)
```

| transcript                                                             |
| ---------------------------------------------------------------------- |
| I saw someone check into a hotel on August 13. The guy looked nervous. |
| I heard someone checked into a hotel with "Sunset" in the name.        |
From this we can discern the details that 19860813 would be our date; and the hotel name would contain Sunset. In order to account for Something like The Sunset Hotel, we can wrap the text in `%` as shown below.
```sql
select * from hotel_checkins where check_in_date = 19860813 and hotel_name like "%Sunset%"
```

However, this yields a lot of results... what can we do? well we see that the transcript states "The guy looked nervous," so maybe the hotel surveillance records can help us narrow this down, but that's a large list... while we *can* generate a csv and utilize that, we can utilize a powerful kit in the SQL bag: JOIN, `INNER JOIN` allows us to join tables by matching up specific columns. We see that the surveillance_records has a hotel_checkin_id column, so let's try matching those up with our existing information.

```mysql
select * from hotel_checkins 
left join surveillance_records on hotel_checkins.id = hotel_checkin_id
where check_in_date = 19860813 and hotel_name like "%Sunset%" and suspicious_activity NOT null
```

> [!note]
> adding the "NOT null" condition ensures that we don't end up with ids where there was no suspicious activity.

| id  | person_id | suspicious_activity            |
| --- | --------- | ------------------------------ |
| 78  | 3         | Used the hotel gym             |
| 34  | 6         | Spotted entering late at night |
| 2   | 8         | Left suddenly at 3 AM          |
| 44  | 15        | Asked for directions to beach  |
| 38  | 20        | Used concierge service         |
| 22  | 29        | Used hotel restaurant          |
| 92  | 36        | Used hotel parking             |
| 54  | 39        | Used ice machine               |
| 94  | 45        | Asked about check-in time      |
| 14  | 48        | Requested room cleaning        |
| 36  | 50        | Used hotel elevator            |
| 58  | 53        | Used hotel gym                 |
| 28  | 57        | Used hotel restaurant          |
| 90  | 60        | Used vending machine           |
| 42  | 74        | Used hotel parking             |
| 24  | 76        | Requested wake-up call         |
| 32  | 81        | Used business center           |
| 18  | 83        | Requested extra towels         |
| 99  | 87        | Asked about food service       |
| 46  | 88        | Used valet service             |
| 26  | 90        | Requested room cleaning        |
| 7   | 92        | Used hotel gym                 |
| 64  | 96        | Used hotel restaurant          |
| 86  | 98        | Requested extra pillows        |
| 70  | 100       | Used concierge service         |
| 78  | 103       | Used the spa facilities        |
| 34  | 106       | Used the business center       |
| 2   | 108       | Asked about local tours        |
| 44  | 115       | Asked for extra blankets       |
| 38  | 120       | Used valet parking             |
| 22  | 129       | Asked for extra towels         |
| 92  | 136       | Asked about local attractions  |
| 54  | 139       | Requested extra pillows        |
| 94  | 145       | Used the pool                  |
| 14  | 148       | Used valet parking             |
| 36  | 150       | Asked for directions           |
| 58  | 153       | Requested wake-up call         |
| 28  | 157       | Broke the vending machine      |
| 90  | 160       | Requested room cleaning        |
| 42  | 174       | Requested airport shuttle      |
| 24  | 176       | Used the fitness center        |
| 32  | 181       | Requested room service         |
| 18  | 183       | Used the sauna                 |
| 99  | 187       | Used the bar                   |
| 46  | 188       | Requested taxi service         |
| 26  | 190       | Used the gym                   |
| 7   | 192       | Asked for directions           |
| 64  | 196       | Requested wake-up call         |
| 86  | 198       | Used the business center       |
| 70  | 200       | Asked about checkout time      |
still quite a lengthy list, but getting somewhere. We can use another join statement to add a 3rd table to the mix! confessions.


## Retrieving the confession

> [!note]
> the conditional clause *MUST* go at the end otherwise it comes up with an error. 
```mysql
select hotel_checkins.id, surveillance_records.person_id, surveillance_records.suspicious_activity, confessions.confession from hotel_checkins 
inner join surveillance_records on hotel_checkins.id = hotel_checkin_id
inner join confessions on surveillance_records.person_id = confessions.person_id
where check_in_date = 19860813 and hotel_name like "%Sunset%" and suspicious_activity NOT null
```

| id  | person_id | suspicious_activity            | confession                                                                 |
| --- | --------- | ------------------------------ | -------------------------------------------------------------------------- |
| 34  | 6         | Spotted entering late at night | I don't know anything about this.                                          |
| 2   | 8         | Left suddenly at 3 AM          | Alright! I did it. I was paid to make sure he never left the marina alive. |
| 44  | 15        | Asked for directions to beach  | This is ridiculous, I'm innocent!                                          |
| 38  | 20        | Used concierge service         | I don't know what you're investigating.                                    |
| 22  | 29        | Used hotel restaurant          | I was at the movies that evening.                                          |
| 92  | 36        | Used hotel parking             | This is all a mistake.                                                     |
| 54  | 39        | Used ice machine               | You're barking up the wrong tree.                                          |
| 94  | 45        | Asked about check-in time      | Check my records, I'm clean.                                               |
| 14  | 48        | Requested room cleaning        | You're making a big mistake.                                               |
| 36  | 50        | Used hotel elevator            | I was at work all day.                                                     |

This is a shortened list, but we got lucky that it was near the top, so we don't need to do any more digging. now to see who this mysterious person_id = 8 is...

```mysql
select name from persons where id = 8
```

| name         |
| ------------ |
| Thomas Brown |
another way is to simply add another join statement which just joins the `person.id` and any one of the `tables.person_id`. I'll leave that as an exercise to the reader
![Case_Solved](../images/Case_Solved.png)

# Closing thoughts

This is the first of the advanced investigations, and tests your knowledge of more advanced querying like `join` where you can connect the dots between tables based on certain criteria

Stay Savvy Detectives.
```
  .OOOOOOOOOOOOOOO @@         D i c k  T r a c y        @@ OOOOOOOOOOOOOOOO.
  OOOOOOOOOOOOOOOO @@                                    @@ OOOOOOOOOOOOOOOO
  OOOOOOOOOO'''''' @@                                    @@ ```````OOOOOOOOO
  OOOOO'' aaa@@@@@@@@@@@@@@@@@@@@"""                   """""""""@@aaaa `OOOO
  OOOOO,""""@@@@@@@@@@@@@@""""                                     a@"" OOOA
  OOOOOOOOOoooooo,                                            |OOoooooOOOOOS
  OOOOOOOOOOOOOOOOo,            I'll be selling the           |OOOOOOOOOOOOC
  OOOOOOOOOOOOOOOOOO            house and moving my          ,|OOOOOOOOOOOOI
  OOOOOOOOOOOOOOOOOO @           family to a condo           |OOOOOOOOOOOOOI
  OOOOOOOOOOOOOOOOO'@           complex with a pool          OOOOOOOOOOOOOOb
  OOOOOOOOOOOOOOO'a'            & where someone else         |OOOOOOOOOOOOOy
  OOOOOOOOOOOOOO''              mows the grass!           aa`OOOOOOOOOOOP
  OOOOOOOOOOOOOOb,..            Things here will be           `@aa``OOOOOOOh
  OOOOOOOOOOOOOOOOOOo           hectic for the next             `@@@aa OOOOo
  OOOOOOOOOOOOOOOOOOO|          6 weeks or so.                     @@@ OOOOe
  OOOOOOOOOOOOOOOOOOO@                               aaaaaaa       @@',OOOOn
  OOOOOOOOOOOOOOOOOOO@                        aaa@@@@@@@@""        @@ OOOOOi
  OOOOOOOOOO~~ aaaaaa"a                 aaa@@@@@@@@@@""            @@ OOOOOx
  OOOOOO aaaa@"""""""" ""            @@@@@@@@@@@@""               @@@|`OOOO'
  OOOOOOOo`@@a                  aa@@  @@@@@@@""         a@        @@@@ OOOO9
  OOOOOOO'  `@@a               @@a@@   @@""           a@@   a     |@@@ OOOO3
  `OOOO'       `@    aa@@       aaa"""          @a        a@     a@@@',OOOO'

```
(credit Roy Sussman for the ascii)
