# Setting the Stage

We are given this tidbit of info about potential corporate espionage!
```txt
QuantumTech, Miami’s leading technology corporation, was about to unveil its groundbreaking microprocessor called “QuantaX.” Just hours before the reveal, the prototype was destroyed, and all research data was erased. Detectives suspect corporate espionage.
```

## Initial info
- Company: QuantumTech
- crime: destruction and data erasure

## The databases

- incident_reports

| Column      | Type    |
| ----------- | ------- |
| id          | INTEGER |
| date        | INTEGER |
| location    | TEXT    |
| description | TEXT    |

- witness_statements

| Column      | Type    |
| ----------- | ------- |
| id          | INTEGER |
| incident_id | INTEGER |
| employee_id | INTEGER |
| statement   | TEXT    |

- keycard_access_logs

| Column       | Type    |
| ------------ | ------- |
| id           | INTEGER |
| employee_id  | INTEGER |
| keycard_code | TEXT    |
| access_date  | INTEGER |
| access_time  | TEXT    |

- computer_access_logs

| Column          | Type    |
| --------------- | ------- |
| id              | INTEGER |
| employee_id     | INTEGER |
| server_location | TEXT    |
| access_date     | INTEGER |
| access_time     | TEXT    |

- email_logs

| Column                | Type    |
| --------------------- | ------- |
| id                    | INTEGER |
| sender_employee_id    | INTEGER |
| recipient_employee_id | INTEGER |
| email_date            | INTEGER |
| email_subject         | TEXT    |
| email_content         | TEXT    |

- facility_access_logs

| Column        | Type    |
| ------------- | ------- |
| id            | INTEGER |
| employee_id   | INTEGER |
| facility_name | TEXT    |
| access_date   | INTEGER |
| access_time   | TEXT    |

- employee_records

| Column        | Type    |
| ------------- | ------- |
| id            | INTEGER |
| employee_name | TEXT    |
| department    | TEXT    |
| occupation    | TEXT    |
| home_address  | TEXT    |

# Investigation
## The incident
First we must figure out the incident so for that we need the location and description of the incident.

```mysql
select * from incident_reports where location like "%QuantumTech%";
```

|id|date|location|description|
|---|---|---|---|
|74|19890421|QuantumTech HQ|Prototype destroyed; data erased from servers.|

Luckily this query only yielded one result, now we have a bit more intel to work off of.

## Intel
- date: 19890421
- location: QuantumTech HQ (there could be more)
- incident_id: 74

## Gathering the statements
According to our database schemas, we have witness statements that correlate to an incident id;

```mysql
select statement from witness_statements where incident_id = 74;
```

| statement                                                                       |
| ------------------------------------------------------------------------------- |
| I heard someone mention a server in Helsinki.                                   |
| I saw someone holding a keycard marked QX- succeeded by a two-digit odd number. |

Now we have some more information

### New Intel
- server: Helsinki
- keycard "QX-" and is only two-digits

```mysql
select computer_access_logs.employee_id, computer_access_logs.access_time computer_time, keycard_access_logs.keycard_code, keycard_access_logs.access_time keycard_time
from computer_access_logs  
inner join keycard_access_logs on keycard_access_logs.employee_id = computer_access_logs.employee_id
where server_location = "Helsinki" and computer_access_logs.access_date = 19890421
```

|employee_id|computer_time|keycard_code|keycard_time|
|---|---|---|---|
|99|09:00|QX-035|08:30|

>[!note]
>when doing a select query, to keep things unambiguous we can use substitute names, for instance "Computer_access_logs.access_time computer_time" renames the column to "computer_time" when outputting, reducing ambiguity about which column goes where.

## Digging Deeper

We see that there was computer access at 09:00 and keycard usage 30 minutes prior. And that the keycard of this employee matches the code provided by a witness, QX-035 ("QX-" followed by a 2 digit odd number). So we seem to be onto something, can we maybe dig up info on if they may have sent any emails?

```mysql
select * from email_logs where sender_employee_id = 99 or recipient_employee_id = 99;
```

|id|sender_employee_id|recipient_employee_id|email_date|email_subject|email_content|
|---|---|---|---|---|---|
|126|263|99|19890421|Alarm System Concern|I noticed something strange with the alarm system. There might be a potential malfunction near the chip. Thought you should check it out to be safe.|

Well, we see that the email is dated for 19890421, let's see if there's more from this employee 263!

```mysql
select * from email_logs where (sender_employee_id = 263 or recipient_employee_id = 263) and recipient_employee_id != 99
```


| id  | sender_employee_id | recipient_employee_id | email_date | email_subject            | email_content                                                                                                                                                                              |
| --- | ------------------ | --------------------- | ---------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 138 | NULL               | 263                   | 19890421   | Realign Asset Trajectory | L’s schedule puts her close enough, but we need her inside F18 before 9. Trigger a minor alert or routine checkup to send her in by 8:30. Make sure she logs the visit. That part matters. |
| 140 | NULL               | 263                   | 19890421   | Execute Phase Window     | Unlock 18 quietly by 9. He’ll use his own credentials to access it shortly after L leaves. No questions. Just ensure the timing lines up. The trail will lead exactly where it needs to.   |

Well, no sender id doesn't bode well, but this could be a contact outside of the company, so no ID is mapped... now; we see that the email mentions someone named "L" but that's no good... (we do know that employee 99 was involved in a security alert... so maybe they're "L" but that seems a minor detail), but there's a mention of "18" or "F18" in the emails as well, alongside times... maybe we can utilize that to our advantage. 

```mysql 
select employee_id, access_date, access_time from facility_access_logs where facility_name like "F%18"
```

|employee_id|access_date|access_time|
|---|---|---|
|290|19890421|12:56|
|99|19890421|08:55|
|297|19890421|09:01|

## Closing the Case
well looks like our mysterious employee 99 appears again... but they're being used as a pawn... but it does seem like there is an access time around 9:01 from a mysterious 297 which lines up with the timing as stated in both the emails. so let's round up our employees.

```mysql
select * from employee_records where id in (99,263,297)
```

| id  | employee_name    | department        | occupation               | home_address                              |
| --- | ---------------- | ----------------- | ------------------------ | ----------------------------------------- |
| 99  | Elizabeth Gordon | Engineering       | Solutions Architect      | 147 Coastal Pine Rd, Doral, FL            |
| 263 | Norman Owens     | Quantum Computing | Quantum Systems Engineer | 234 Quantum Waters Lane, Key Biscayne, FL |
| 297 | Hristo Bogoev    | Engineering       | Principal Engineer       | 901 Quantum Ocean Way, Key Biscayne, FL   |

So it appears that the mysterious 'L' is the Solutions Architect, Elizabeth Gordon. She was being framed by the Principal Engineer, Hristo Bogoev, and the Principal Engineer, Norman Owens. And as it would appear by the solution being accepted that Hristo was the one who pulled the trigger (Hmm, I wonder why that name sounds familiar.)


# Closing Thoughts
This case had us diving through multiple different hoops to land on the right culprit, requiring us to put on our thinking caps and investigate the very narrow clues we were granted. But by being able to zero in on the key details, like the Helsinki server and keycard value, at the start it allowed us to gain deeper knowledge of who was behind it.


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