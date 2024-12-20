**Schema (PostgreSQL v17)**

    DROP TABLE IF EXISTS children CASCADE;
    DROP TABLE IF EXISTS wish_lists CASCADE;
    DROP TABLE IF EXISTS toy_catalogue CASCADE;
    
    CREATE TABLE children (
      child_id INT PRIMARY KEY,
      name VARCHAR(100),
      age INT
    );
    CREATE TABLE wish_lists (
        list_id INT PRIMARY KEY,
        child_id INT,
        wishes JSON,
        submitted_date DATE
    );
    CREATE TABLE toy_catalogue (
        toy_id INT PRIMARY KEY,
        toy_name VARCHAR(100),
        category VARCHAR(50),
        difficulty_to_make INT
    );
    
    INSERT INTO children VALUES
    (1, 'Elinor', 10),
    (2, 'Yoshiko', 9),
    (3, 'Alta', 14),
    (4, 'Evalyn', 5),
    (5, 'Sim', 10),
    Etc......
    
    INSERT INTO wish_lists VALUES
    (1, 160, '{"colors":["White","Brown","Blue"],"first_choice":"Toy kitchen sets","second_choice":"Barbie dolls"}', '2024-05-11'),
    (2, 97, '{"colors":["Orange"],"first_choice":"LEGO blocks","second_choice":"Building sets"}', '2024-02-05'),
    (3, 548, '{"colors":["Brown","Purple","White","Yellow","Purple"],"first_choice":"LEGO blocks","second_choice":"Toy trains"}', '2024-06-18'),
    (4, 978, '{"colors":["Red","Purple"],"first_choice":"Building sets","second_choice":"Rubiks cubes"}', '2024-03-10'),
    (5, 983, '{"colors":["Yellow","Blue"],"first_choice":"Toy trucks","second_choice":"Stuffed animals"}', '2024-10-13'),
    Etc.....
    
    INSERT INTO toy_catalogue VALUES
    (1, 'LEGO blocks', 'educational', 3),
    (2, 'Teddy bears', 'indoor', 4),
    (3, 'Dollhouses', 'indoor', 5),
    (4, 'Remote control cars', 'indoor_outdoor', 4),
    (5, 'Action figures', 'indoor', 3),
    (6, 'Jigsaw puzzles', 'educational', 2),
    (7, 'Play-Doh', 'educational', 1),
    (8, 'Building blocks', 'educational', 2),
    (9, 'Toy trains', 'indoor', 3),
    (10, 'Jump ropes', 'outdoor', 1),
    (11, 'Board games', 'indoor', 3),
    (12, 'Toy kitchen sets', 'indoor', 4),
    (13, 'Stuffed animals', 'indoor', 3),
    (14, 'Barbie dolls', 'indoor', 2),
    (15, 'Yo-yos', 'indoor_outdoor', 1),
    (16, 'Building sets', 'educational', 4),
    (17, 'Toy trucks', 'indoor_outdoor', 2),
    (18, 'Hula hoops', 'outdoor', 1),
    (19, 'Rubiks cubes', 'educational', 3),
    (20, 'Toy musical instruments', 'educational', 3);
    

---

**Query #1**

    with flat as (
    select * 
    	, wishes->'colors'->>0 as favourite_colour
        , wishes->'first_choice' as primary_choice
        , wishes->'second_choice' as secondary_choice
        , wishes->'colors' AS colours
      	, json_array_length(wishes->'colors') AS colour_count
    from wish_lists
      )
      
     , clean as (
      select list_id
        , child_id
        , favourite_colour
        , replace(primary_choice::varchar, '"', '') as primary_choice
       	, replace(secondary_choice::varchar, '"', '') as backup_choice
       	, colour_count
      from flat
     )
     
     , stage as (
     select c.* 
     	, k.name
        , t.category
        , t.difficulty_to_make
     from clean as c
     inner join toy_catalogue as t
     	on t.toy_name = c.primary_choice
     inner join children as k
     	on k.child_id = c.child_id
     )
    
    select name
    	, primary_choice
        , backup_choice
        , favourite_colour
        , colour_count
        , case 
        	when difficulty_to_make = 1 then 'Simple Gift'
            when difficulty_to_make = 2 then 'Moderate Gift'
            when difficulty_to_make >= 3 then 'Complex Gift'
            	end as gift_complexity
        , case 
       		when category = 'outdoor' then 'Outside Workshop'
            when category = 'educational' then 'Learning Workshop'
            else 'General Workshop'
            	end as workshop_assignment
     from stage
     order by name asc
     limit 5

| name    | primary_choice  | backup_choice   | favourite_colour | colour_count | gift_complexity | workshop_assignment |
| ------- | --------------- | --------------- | ---------------- | ------------ | --------------- | ------------------- |
| Abagail | Building sets   | LEGO blocks     | Blue             | 1            | Complex Gift    | Learning Workshop   |
| Abbey   | Stuffed animals | Teddy bears     | White            | 4            | Complex Gift    | General Workshop    |
| Abbey   | Barbie dolls    | Play-Doh        | Purple           | 1            | Moderate Gift   | General Workshop    |
| Abbey   | Toy trains      | Toy trains      | Pink             | 2            | Complex Gift    | General Workshop    |
| Abbey   | Yo-yos          | Building blocks | Blue             | 5            | Simple Gift     | General Workshop    |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/8WVgob4sMV2iiymgPPmLg1/0)