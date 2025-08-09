# Setting The Scene
We are given the following text to set the mood:

```txt
In the neon glow of 1980s Los Angeles, the West Hollywood Records store was rocked by a daring theft. A prized vinyl record, worth over $10,000, vanished during a busy evening, leaving the store owner desperate for answers. Vaguely recalling the details, you know the incident occurred on July 15, 1983, at this famous store. Your task is to track down the thief and bring them to justice.
```

## Initial information
- Location: West HollyWood Records
- Type: Theft
- Item: Vinyl Record
- date: 19830715 (dates are in yyyymmdd format)

## The databases
1. crime_scene
	- id
	- date
	- type
	- location
	- description
2. witnesses
	- id
	- crime_scene_id
	- clue
3. suspects
	- id
	- name
	- bandana_color
	- accessory
4. interviews
	- suspect_id
	- transcript

# The investigation

## Crime Scene Start
As any good detective would do, we search up the case information.

```sql
select * from crime_scene where location = "West Hollywood Records" and date = 19830715;
```

This doesn't return much new information, but we do know from our witnesses schema table, that we can use the crime scene id -- which is 65 to sniff out any leads.

## Checking the Witness Statements

Since it doesn't appear the witness id would be too useful we can just select the clue column from the "witnesses" database

```sql
select clue from witnesses where crime_scene_id = 65;
```

This query yields the following information:

| clue                                                                            |
| ------------------------------------------------------------------------------- |
| I saw a man wearing a red bandana rushing out of the store.                     |
| The main thing I remember is that he had a distinctive gold watch on his wrist. |
This gives us two new important information if we reference our table information:
- bandana_color = "red"
- accessory = "gold watch"

## Narrowing down the suspects
Let's formulate a query for the "suspects" database that takes into consideration the bandana color and accessory.

```sql
select id,name from suspects where bandana_color = "red" and accessory = "gold watch";
```

| id  | name          |
| --- | ------------- |
| 35  | Tony Ramirez  |
| 44  | Mickey Rivera |
| 97  | Rico Delgado  |
This returns 3 names and ids, here I'll show a neat trick I picked up from the more advanced cases where you may have *way* more information.

```sql
select string_agg(id,","),string_agg(name," - ")from suspects where bandana_color = "red" and accessory = "gold watch";
```

`string_agg(column,"delimiter")` concatenates (combines) the column values and separates them by the delimiter specified, the delimiter can be multiple characters, as shown in `string_agg(name," - ")`. Since none of the other tables use the names, you don't necessarily need to include the name column in this, but it can be useful.

| string_agg(id,",") | string_agg(name," - ")                      |
| ------------------ | ------------------------------------------- |
| 35,44,97           | Tony Ramirez - Mickey Rivera - Rico Delgado |
## Retrieving the confession
Now that we have a comma separated list we can use that to find out some information from the "interviews" table.

```sql
select * from interviews where suspect_id in (35,44,97);
```

|suspect_id|transcript|
|---|---|
|35|I wasn't anywhere near West Hollywood Records that night.|
|44|I was busy with my music career; I have nothing to do with this theft.|
|97|I couldn't help it. I snapped and took the record.|
Now we have a confession and suspect_id; let's cross reference that with our suspects that we discovered earlier -- we see that the person whodunnit is.... Rico

> [!note] 
> The answer field is expecting the full name, not partial. 

![Case_Solved](../images/Case_Solved.png)

# Final Thoughts on the case
Through careful cross-referencing of data, we were able to narrow down the suspects in no time thus reinforcing that we should always be looking for the most amount of correlational data when we are doing our searches.

Thanks for joining me on the case detectives, stay savvy
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
