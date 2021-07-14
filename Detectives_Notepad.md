# Detective's Notepad

> This notepad may contain spoilers. This is for personal use along with the project, and should not be referenced unless you want to follow my train of thought.


### Steps

1. Locate the lost crime scene report
``` sql
    SELECT * FROM crime_scene_report
    WHERE type = 'murder'
    AND date = 20180115
    AND city = 'SQL City';
```

| date | type | description | city |
| ---- | ---- | ----------- | ---- |
| 20180115 | murder | Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave". | SQL City

---

2. Identify the Witnesses

``` sql
    SELECT *, MAX(address_number) FROM person
    WHERE address_street_name = "Northwestern Dr";
```
| id | name | license_id | address_number | address_street_name | ssn |
| --- | --- | --- | --- | --- | --- |
| 14887 | Morty Schapiro | 118009 | 4919 | Northwestern Dr | 111564949 |


``` sql
    SELECT * FROM person
    WHERE address_street_name = "Franklin Ave"
    AND name LIKE "%Annabel%";
```
| id | name | license_id | address_number | address_street_name | ssn |
| --- | --- | --- | --- | --- | --- |
| 16371 | Annabel Miller | 490173 | 103 | Franklin Ave | 318771143 |

---

3. Interview the Witnesses

``` sql
    SELECT * FROM interview
    WHERE person_id = 14887;
```
| person_id | transcript |
| --- | --- |
| 14887 | I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W". |


``` sql
    SELECT * FROM interview
    WHERE person_id = 16371;
```
| person_id | transcript |
| --- | --- |
| 14887 | I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th. |

---

4. Narrow Down the Gym Members

``` sql
    SELECT  id, name, check_in_date, membership_status
    FROM get_fit_now_member, get_fit_now_check_in
    WHERE get_fit_now_member.id = get_fit_now_check_in.membership_id
    AND get_fit_now_member.membership_status = 'gold'
    AND get_fit_now_member.id LIKE '48Z%'
    AND get_fit_now_check_in.check_in_date = '20180109';
```
| id | name | check_in_date | membership_status |
| --- | --- | --- | --- |
| 48Z7A | Joe Germuska | 20180109 | gold |
| 48Z55 | Jeremy Bowers | 20180109 | gold |

---

5. Narrow Down the Driver

``` sql
    SELECT person.license_id, plate_number, name from drivers_license, person
    WHERE plate_number LIKE '%H42W%'
    AND drivers_license.id = person.license_id;
```
| license_id | plate_number | name |
| --- | --- | --- |
| 664760 | 4H42WR | Tushar Chandra |
| 423327 | 0H42W2 | Jeremy Bowers |
| 183779 | H42W0X | Maxine Whitely |
---

6. Investigate the Suspects

``` sql
    SELECT * FROM person
    WHERE name = "Joe Germuska"
    OR name = "Jeremy Bowers";
```
| id | name | license_id | address_number | address_street_name | ssn |
| --- | --- | --- | --- | --- | --- |
| 28819 | Joe Germuska | 173289 | 111 | Fisk Rd | 138909730 |
| 67318 | Jeremy Bowers | 423327 | 530 | Washington Pl, Apt 3A | 871539279 |

``` sql
    SELECT interview.person_id, interview.transcript, person.name
    FROM interview
    JOIN person on interview.person_id = person.id
    WHERE person.name = 'Jeremy Bowers'
    OR person.name = 'Joe Germuska';
```

| id | transcript | name |
| --- | --- | --- |
| 67318 | I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017. | Jeremy Bowers |

---

7. Search for the Cars

``` sql
    SELECT id, height, hair_color, car_model from drivers_license
    WHERE height BETWEEN 65 AND 67
    AND car_model = 'Model S'
    AND car_make = 'Tesla'
    AND hair_color = 'red';
```

| id | height | hair_color | car_model |
| --- | --- | --- | --- |
| 202298 | 66 | red | Model S |
| 291182 | 66 | red | Model S |
| 918773 | 65 | red | Model S |

---

8. Identify the Owners

``` sql
    SELECT * FROM person
    WHERE license_id = '202298'
    OR license_id = '291182'
    OR license_id = '918773';
```

| id | name | license_id | address_number | address_street_name | ssn |
| --- | --- | --- | --- | --- | --- |
| 78881 | Red Korb | 918773 | 107 | Camerata Dr | 961388910 |
| 90700 | Regina George | 291182 | 332 | Maple Ave | 337169072 |
| 99716 | Miranda Priestly | 202298 | 1883 | Golden Ave | 987756388 |

---

9. Cross-reference their Schedule

``` sql
    SELECT person_id, event_name, date from facebook_event_checkin fec
    JOIN person p on fec.person_id = p.id
    WHERE date LIKE '2017%'
    AND person_id = 78881
    OR person_id = 90700
    OR person_id = 99716;
```

| person_id | event_name | date |
| --- | --- | --- |
| 99716 | SQL Symphony Concert | 20171206 |
| 99716 | SQL Symphony Concert | 20171212 |
| 99716 | SQL Symphony Concert | 20171229 |

---

10. Verify the Suspects Income Level

``` sql
    SELECT * from income
    WHERE ssn = 987756388;
```

| ssn | annual_income |
| --- | --- |
| 987756388 | 310000 |

---
