# Setting the Scene
We're given the following text to start off our hunt.

```txt
On October 31, 1987, at a Coconut Grove mansion masked ball, Leonard Pierce was found dead in the garden. Can you piece together all the clues to expose the true murderer?
```

## Initial Info
- date: 19871031
- location: Coconut Grove Mansion
- crime: murder

## The databases
- crime_scene:
  
| Column | Type    |
| ------ | ------- |
| id     | INTEGER |
|date|INTEGER|
|location|TEXT|
|description|TEXT|
  
- person:
  
| Column     | Type    |
| ---------- | ------- |
| id         | INTEGER |
| name       | TEXT    |
| occupation | TEXT    |
| address    | TEXT    |
  
- witness_statements:

| Column         | Type    |
| -------------- | ------- |
| id             | INTEGER |
| crime_scene_id | INTEGER |
| witness_id     | INTEGER |
| clue           | TEXT    |

- hotel_checkins:

| Column        | Type    |
| ------------- | ------- |
| id            | INTEGER |
| person_id     | INTEGER |
| hotel_name    | TEXT    |
| check_in_date | INTEGER |
| room_number   | TEXT    |

- surveillance_records:

| Column           | Type    |
| ---------------- | ------- |
| id               | INTEGER |
| hotel_checkin_id | INTEGER |
| note             | TEXT    |

- phone_records:

| Column       | Type    |
| ------------ | ------- |
| id           | INTEGER |
| caller_id    | INTEGER |
| recipient_id | INTEGER |
| call_date    | INTEGER |
| call_time    | TEXT    |
| note         | TEXT    |

- final_interviews:

| Column     | Type    |
| ---------- | ------- |
| id         | INTEGER |
| person_id  | INTEGER |
| confession | TEXT    |

- vehicle_registry:

| Column       | Type    |
| ------------ | ------- |
| id           | INTEGER |
| person_id    | INTEGER |
| plate_number | TEXT    |
| car_make     | TEXT    |
| car_model    | TEXT    |

- catering_orders:

| Column     | Type    |
| ---------- | ------- |
| id         | INTEGER |
| person_id  | INTEGER |
| order_date | INTEGER |
| item       | TEXT    |
| amount     | INTEGER |

# The Investigation 
## The Crime Scene

```mysql
select * from crime_scene where date = 19871031 and location like "%Coconut Grove%"
```

| id  | date     | location                     | description                                                                                                              |
| --- | -------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| 75  | 19871031 | Miami Mansion, Coconut Grove | During a masked ball, a body was found in the garden. Witnesses mentioned a hotel booking and suspicious phone activity. |


### Intel
- Hotel booking
- Suspicious phone activity

## Digging further
Well, our crime scene description isn't much to go off of to start with... but we can work with that to start. We can use some words that are similar to "booking" I chose a common word, reservation. for my query.

```mysql
select clue from witness_statements where clue like "%booking%" or clue like "%reservation%" and crime_scene_id = 75
```

|clue|
|---|
|I overheard a booking at The Grand Regency.|
|I noticed someone at the front desk discussing Room 707 for a reservation made yesterday.|

### New Intel

- Hotel Name: The Grand Regency
- Room: 707
- Reservation date: 19871030

What can we learn from this, time to dig in to hotel_checkins and see what we can find

```mysql
select hotel_checkins.id, hotel_checkins.person_id, surveillance_records.note
from hotel_checkins 
inner join surveillance_records on surveillance_records.hotel_checkin_id = hotel_checkins.id
where hotel_name = "The Grand Regency" and check_in_date = 19871030 and room_number = 707 and note NOT null 
```


|id|person_id|note|
|---|---|---|
|78|78|Subject attended hotel sponsored cooking demonstration in restaurant.|
|109|34|Subject used hotel shuttle service to shopping mall.|
|119|11|Subject was overheard yelling on a phone: "Did you kill him?"|
|183|198|Subject participated in hotel guest mixer event.|
|188|123|Subject used hotel business center printer.|
|193|156|Subject used hotel shoe shine service.|

Now we're getting juicy! we see that on the surveillance_records that person 11 was heard asking "Did you kill him?" Now, let's see if we can't dive into those phone logs from the event!

|caller_id|recipient_id|note|
|---|---|---|
|11|58|Why did you kill him, bro? You should have left the carpenter do it himself!|

Now we have an Occupation, which can be cross referenced with persons of interest

```mysql
select * from person where occupation = "Carpenter"
```

|id|name|occupation|address|
|---|---|---|---|
|51|Frank Price|Carpenter|939 Hemlockwood Avenue|
|90|Julie Sanders|Carpenter|345 Juniperwood Way|
|97|Marco Santos|Carpenter|112 Forestwood Way|
|134|Amy Evans|Carpenter|223 Redwood Road|
|176|Judith Fisher|Carpenter|889 Redwood Road|

well that's a few people... let's see if we can find something from this using a handy dandy join statement!

```mysql
select person.name,final_interviews.confession 
from person 
inner join final_interviews on final_interviews.person_id = person.id
where occupation = "Carpenter"
```

| name          | confession                                                             |
| ------------- | ---------------------------------------------------------------------- |
| Frank Price   | Youre making a mistake. I didnt kill that person.                      |
| Julie Sanders | I was visiting my parents. I couldnt possibly kill someone.            |
| Marco Santos  | I ordered the hit. It was me. You caught me.                           |
| Amy Evans     | Check my internet service logs. Im not the murderer youre looking for. |
| Judith Fisher | The bank cameras caught me making a deposit. I wouldnt take a life.    |


Et voila, my dear Holmes you've cracked another case! It was Marco who ordered the hit on Leonard Pierce, a Masquerade makes for a decent cover for a murder... if only the surveillance footage didn't catch them...

There is another way to figure out it was Marco, considering we have a call log with two other persons of interest... but that's an exercise for the reader.

![Case_Solved](../images/Case_Solved.png)

# Final Thoughts
Well detectives, it was another tough day in the office. But we cracked the case of whodunnit. And as we can see, sometimes there may be red herring databases which lead us nowhere. But, using our solid sleuthing skills we were able to track down who ordered the kill.

Stay Savvy Detectives.

---
```ascii
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