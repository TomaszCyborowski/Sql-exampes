# SQL

-- Unpivot using cross apply
select id, VarName, value
from ExampleTable cross apply
( values ('Name', Name),('Description',Description)) tt (VarName, value);
 
select id,varname, value from ExampleTable
Unpivot
(
  value for varname in (Name, Description)
) as UnPvt

ECLARE @Result VARCHAR(8000)

-- Using xml to split string
SELECT @Result = COALESCE(@Result + ', ', '') + Sentence.Data
FROM
(
      SELECT Split.a.value('.', 'VARCHAR(8000)') AS Data 
      FROM 
      (
            SELECT CAST ('<M>' + REPLACE('I need a password containing no pass-phrase. Please help.', ' ', '</M><M>') + '</M>' AS XML) AS Data      
      ) AS A CROSS APPLY Data.nodes ('/M') AS Split(a)
)AS Sentence
INNER JOIN
(
      SELECT Split.a.value('.', 'VARCHAR(8000)') AS Data 
      FROM 
      (
            SELECT CAST ('<M>' + REPLACE('word phrase', ' ', '</M><M>') + '</M>' AS XML) AS Data      
) AS A CROSS APPLY Data.nodes ('/M') AS Split(a)
) AS Word ON PATINDEX('%'+Word.Data+'%',Sentence.Data )>0

SELECT @Result

-- recursive CTE
declare @Users as table
    (UserId  int)
;

declare @Relations as table
    (PrimaryUser  int, SecondaryUser int)
;    

INSERT INTO @Relations
    (PrimaryUser, SecondaryUser)
VALUES
    (1,2),
    (1,3),
    (2,4),
    (2,7),
    (2,8),
    (5,6),
    (6,19)

INSERT INTO @Users
    (UserId)
VALUES
    (1),
    (2),
    (3),
    (4),
    (5),
    (7),
    (5),
    (6),
    (19),
    (20)

;WITH cte1 AS (
  SELECT UserId AS [User]
  FROM @Users 
  WHERE UserId = 5
  GROUP BY UserId
UNION ALL  
  SELECT SecondaryUser  AS [User]
  FROM cte1
  JOIN @Relations t 
    ON t.PrimaryUser = cte1.[User]   
)
SELECT [User] FROM cte1

-- xml use example
WITH Split_Names (ElementID,Element, xmlname)
AS
(
    SELECT ElementID,
    Element,
    CONVERT(XML,'<Names><name>'  
    + REPLACE(Element,'|', '</name><name>') + '</name></Names>') AS xmlname
      FROM [dbo].[func_Split] ('57|0|0|2|||~56|0|0|2|||~55|0|0|2|||~54|0|0|3|4|5|~53|0|0|4|||~52|0|0|4|||~51|0|0|2|||~','~') 
)

 SELECT Element,      
 xmlname.value('/Names[1]/name[1]','varchar(100)') AS ID,    
 xmlname.value('/Names[1]/name[2]','varchar(100)') AS Element1,    
 xmlname.value('/Names[1]/name[3]','varchar(100)') AS Element2,    
 xmlname.value('/Names[1]/name[4]','varchar(100)') AS Element3,    
 xmlname.value('/Names[1]/name[5]','varchar(100)') AS Element4,    
 xmlname.value('/Names[1]/name[6]','varchar(100)') AS Element5    
 INTO #tmp
 FROM Split_Names

 SELECT Element3 FROM #tmp WHERE ID = 54
