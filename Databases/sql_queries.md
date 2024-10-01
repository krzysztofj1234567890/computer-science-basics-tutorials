## UPDATE from a SELECT statement

```
UPDATE Per
SET 
Per.PersonCityName=Addr.City, 
Per.PersonPostCode=Addr.PostCode
FROM Persons Per
INNER JOIN
AddressList Addr
ON Per.PersonId = Addr.PersonId
```

```
UPDATE Persons
SET  Persons.PersonCityName=(SELECT AddressList.PostCode
                            FROM AddressList
                            WHERE AddressList.PersonId = Persons.PersonId)
```