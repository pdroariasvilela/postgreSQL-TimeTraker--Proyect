# DDL - Time Tracker

## Create a Database

Create a database using the following ERD for the MVP of Time Tracker

![](https://p-vvf5mjm.t4.n0.cdn.getcloudapp.com/items/d5uNbEL7/ecb4b8de-97cd-49e0-bfed-0734f3368c08.jpg?source=viewer&v=d5cff96bca526bc7837d9e1c383a3cb8)

Some clarifications:

- The `rate` of the user is their cost per hour (in US\$)
- The `total_budget` is the amount of money allocated to one user on one project
  between the start and end date of the project.
- The `closed` attribute of a Project define if a project is open (`false`) or
  closed (`true`).
- The Daily-Log register how many hour each member has invest on a project in
  one day.

1. Create a new database called `timetracker`

```SQL
<INSERT YOUR SQL STATEMENTS HERE>
```

Expected result:

```bash
postgres=# \l

    Name           |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-------------------+----------+----------+-------------+-------------+-----------------------
 ...
 timetracker       | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 ...
(xx rows)
```

2. Create the table `users` following the ERD.
   - id should be auto incremented.
   - No field should be null.
   - `rate` should be grater than or equal to 0.

```SQL
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(50) NOT NULL,
  email VARCHAR(25) NOT NULL UNIQUE,
  role VARCHAR(50) NOT NULL,
  rate INTEGER NOT NULL CHECK (rate >= 0)
);
```

Expected result:

```bash
timetracker=# \d users
                                   Table "public.users"
 Column |         Type          | Collation | Nullable |             Default
--------+-----------------------+-----------+----------+----------------------------------
 id     | integer               |           | not null | nextval('user_id_seq'::regclass)
 name   | character varying(50) |           | not null |
 email  | character varying(25) |           | not null |
 role   | character varying(50) |           | not null |
 rate   | integer               |           | not null |
Indexes:
    "users_pkey" PRIMARY KEY, btree (id)
Check constraints:
    "users_rate_check" CHECK (rate >= 0)
```

3. Create the table `projects` following the ERD
   - id should be auto incremented.
   - No field should be null
   - By default the `closed` attribute should be `false`

```SQL
CREATE TABLE projects (
  id SERIAL PRIMARY KEY,
  name VARCHAR(25) NOT NULL,
  category VARCHAR(25) NOT NULL,
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  closed BOOLEAN NOT NULL DEFAULT false
);
```

Expected result:

```bash
timetracker=# \d projects
                                     Table "public.projects"
   Column   |         Type          | Collation | Nullable |               Default
------------+-----------------------+-----------+----------+-------------------------------------
 id         | integer               |           | not null | nextval('project_id_seq'::regclass)
 name       | character varying(25) |           | not null |
 category   | character varying(25) |           | not null |
 start_date | date                  |           | not null |
 end_date   | date                  |           | not null |
 closed     | boolean               |           | not null | false
Indexes:
    "projects_pkey" PRIMARY KEY, btree (id)
```

## Create relationships

4. Create the `projects_users` table.
   - id should be auto incremented.
   - `total_budget` should be grater than or equal to 0.

```SQL
 CREATE TABLE projects_users (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id),
  project_id INTEGER NOT NULL REFERENCES projects(id),
  total_budget INTEGER NOT NULL CHECK (total_budget >= 0)
);
```

Expected result:

```bash
timetracker=# \d projects_users
                                Table "public.projects_users"
    Column    |  Type   | Collation | Nullable |                  Default
--------------+---------+-----------+----------+--------------------------------------------
 id           | integer |           | not null | nextval('"projects_users_id_seq"'::regclass)
 user_id      | integer |           | not null |
 project_id   | integer |           | not null |
 total_budget | integer |           | not null |
Indexes:
    "projects_users_pkey" PRIMARY KEY, btree (id)
Check constraints:
    "projects_users_total_budget_check" CHECK (total_budget >= 0)
Foreign-key constraints:
    "projects_users_project_id_fkey" FOREIGN KEY (project_id) REFERENCES project(id)
    "projects_users_user_id_fkey" FOREIGN KEY (user_id) REFERENCES "user"(id)
```

5. Create the `daily-logs` table,
   - id should be auto incremented.
   - All fields are required.
   - `hours` should be grater than or equal to 0.

```SQL
CREATE TABLE daily_logs (
  id SERIAL PRIMARY KEY,
  user_project_id INTEGER NOT NULL REFERENCES projects_users(id),
  date DATE NOT NULL,
  hours INTEGER NOT NULL CHECK (hours >= 0)
);
```

Expected result:

```bash
timetracker=# \d daily_logs
                                  Table "public.daily-logs"
     Column      |  Type   | Collation | Nullable |                 Default
-----------------+---------+-----------+----------+-----------------------------------------
 id              | integer |           | not null | nextval('"daily-logs_id_seq"'::regclass)
 user_project_id | integer |           | not null |
 date            | date    |           | not null |
 hours           | integer |           | not null |
Indexes:
    "daily-logs_pkey" PRIMARY KEY, btree (id)
Check constraints:
    "daily-logs_hours_check" CHECK (hours >= 0)
Foreign-key constraints:
    "daily-logs_user_project_id_fkey" FOREIGN KEY (user_project_id) REFERENCES "projects_users"(id)
```

## Alter a database

After some revision, some changes should be done to our database.

6. Changes on the `users` table
   - The`email` field should be unique.
   - The name and role fields length are not enough. Change it to a maximum of 50 characters.

```SQL
ALTER TABLE users
ADD CONSTRAINT us_email_key UNIQUE (email),
ALTER COLUMN name TYPE VARCHAR(50),
ALTER COLUMN role TYPE VARCHAR(50);

```

Expected result:

```bash
timetracker=# \d users
                                   Table "public.users"
 Column |         Type          | Collation | Nullable |             Default
--------+-----------------------+-----------+----------+----------------------------------
 id     | integer               |           | not null | nextval('users_id_seq'::regclass)
 name   | character varying(50) |           | not null |
 email  | character varying(25) |           | not null |
 role   | character varying(50) |           | not null |
 rate   | integer               |           | not null |
Indexes:
    "users_pkey" PRIMARY KEY, btree (id)
    "Users_email_key" UNIQUE CONSTRAINT, btree (email)
Check constraints:
    "users_rate_check" CHECK (rate >= 0)
Referenced by:
    TABLE "projects_users" CONSTRAINT "projects_users_user_id_fkey" FOREIGN KEY (user_id) REFERENCES "user"(id)
```

7. Changes on the `projects` table,
   - `end_date` should be greater than `start_date`

```SQL
ALTER TABLE projects 
ADD CONSTRAINT End_after_start_check CHECK (end_date > start_date); 
```

Expected result:

```bash
timetracker=# \d projects
                                     Table "public.projects"
   Column   |         Type          | Collation | Nullable |               Default
------------+-----------------------+-----------+----------+-------------------------------------
 id         | integer               |           | not null | nextval('projects_id_seq'::regclass)
 name       | character varying(25) |           | not null |
 category   | character varying(25) |           | not null |
 start_date | date                  |           | not null |
 end_date   | date                  |           | not null |
 closed     | boolean               |           | not null | false
Indexes:
    "projects_pkey" PRIMARY KEY, btree (id)
Check constraints:
    "End_after_start_check" CHECK (end_date > start_date)
Referenced by:
    TABLE ""projects_users"" CONSTRAINT "projects_users_project_id_fkey" FOREIGN KEY (project_id) REFERENCES project(id)
```

8. Changes on the `projects_users` table to have unique combination of user_id and project_id

```SQL
ALTER TABLE projects_users
ADD CONSTRAINT projects_users_unique_FKs UNIQUE (user_id, project_id);
```

Expected result:

```bash
timetracker=# \d projects_users
                                Table "public.projects_users"
    Column    |  Type   | Collation | Nullable |                  Default
--------------+---------+-----------+----------+--------------------------------------------
 id           | integer |           | not null | nextval('"projects_users_id_seq"'::regclass)
 user_id      | integer |           | not null |
 project_id   | integer |           | not null |
 total_budget | integer |           | not null |
Indexes:
    "projects_users_pkey" PRIMARY KEY, btree (id)
    "projects_users_unique_FKs" UNIQUE CONSTRAINT, btree (user_id, project_id)
Check constraints:
    "projects_users_total_budget_check" CHECK (total_budget >= 0)
Foreign-key constraints:
    "projects_users_project_id_fkey" FOREIGN KEY (project_id) REFERENCES project(id)
    "projects_users_user_id_fkey" FOREIGN KEY (user_id) REFERENCES "user"(id)
Referenced by:
    TABLE ""daily-logs"" CONSTRAINT "daily-logs_user_project_id_fkey" FOREIGN KEY (user_project_id) REFERENCES "projects_users"(id)
```

9. Changes on the `daily-logs` table to have unique combination of user_project_id and date

```SQL
ALTER TABLE daily_logs
ADD CONSTRAINT "user_project_id-date" unique (user_project_id, date);
```

Expected result:

```bash
timetracker=# \d daily-logs
                                  Table "public.daily-logs"
     Column      |  Type   | Collation | Nullable |                 Default
-----------------+---------+-----------+----------+-----------------------------------------
 id              | integer |           | not null | nextval('"daily-logs_id_seq"'::regclass)
 user_project_id | integer |           | not null |
 date            | date    |           | not null |
 hours           | integer |           | not null |
Indexes:
    "daily-logs_pkey" PRIMARY KEY, btree (id)
    "user_project_id-date unique" UNIQUE CONSTRAINT, btree (user_project_id, date)
Check constraints:
    "daily-logs_hours_check" CHECK (hours >= 0)
Foreign-key constraints:
    "daily-logs_user_project_id_fkey" FOREIGN KEY (user_project_id) REFERENCES "projects_users"(id)
```

## Insert values (to check the result just make a SELECT query 🙂)

While the app gets ready, the team has been tracking their hours manually. Use
[this google document](https://docs.google.com/spreadsheets/d/1KrQXWFzv4Nddss2p3xVC9V_ME5vH3ghN0n7JNhLoVQ8/edit?usp=sharing)
to populate your newly created tables.

10. Insert the `users` data.

```SQL
INSERT INTO users (name, email, role, rate)
VALUES
('Renato','renato@codeable.la','front-end developer senior',30),
('Paty','paty@codeable.la','back-end developer senior',32),
('3','franco@codeable.la','front-end developer junior',15),
('Luis','luis@codeable.la','back-end developer junior',16);
```

11. Insert the `projects` data.

```SQL
INSERT INTO projects (name, category, start_date, end_date)
VALUES
('Shiftme' , 'Business' , '2020-05-13' , '2020-08-11'),
('Line Balancing ', 'Business' , '2020-05-13' , '2020-09-10'),
('Overbooking' , 'Business' , '2020-05-13' , '2020-10-10'),
('Kampu' , 'Sports' , '2020-05-13' , '2020-06-27'),
('Codeable App ', 'Education' , '2020-05-13' , '2020-11-09');
```

12. Insert the `projects_users` data.

```SQL
INSERT INTO "projects_users" (user_id, project_id, total_budget)
VALUES
(1 , 1 , 4320),
(1 , 2 , 8640),
(1 , 3 , 18000),
(2 , 4 , 5760),
(2 , 5 , 23040),
(3 , 1 , 2700),
(3 , 5 , 16200),
(4 , 4 , 2880),
(4 , 5 , 5760),
(4 , 1 , 2880);
```

13. Insert the `daily-logs` data.

```SQL
insert into daily_logs (user_project_id, date, hours) 
values 
(1,'2020-05-13',2),
(1,'2020-05-14',2), 
(1,'2020-05-15',4), 
(1,'2020-05-16',3), 
(1,'2020-05-17',2), 
(2,'2020-05-13',3), 
(2,'2020-05-14',4), 
(2,'2020-05-15',3), 
(2,'2020-05-16',4), 
(2,'2020-05-17',5), 
(3,'2020-05-13',3), 
(3,'2020-05-14',2), 
(3,'2020-05-15',1), 
(3,'2020-05-16',1), 
(3,'2020-05-17',1), 
(4,'2020-05-13',4),
(4,'2020-05-14',2), 
(4,'2020-05-15',0), 
(4,'2020-05-16',0), 
(4,'2020-05-17',0), 
(5,'2020-05-13',4), 
(5,'2020-05-14',6), 
(5,'2020-05-15',8), 
(5,'2020-05-16',8), 
(5,'2020-05-17',8), 
(6,'2020-05-13',5), 
(6,'2020-05-14',2), 
(6,'2020-05-15',4), 
(6,'2020-05-16',4), 
(6,'2020-05-17',3), 
(7,'2020-05-13',3), 
(7,'2020-05-14',6), 
(7,'2020-05-15',4), 
(7,'2020-05-16',4), 
(7,'2020-05-17',5), 
(8,'2020-05-13',2), 
(8,'2020-05-14',2), 
(8,'2020-05-15',0), 
(8,'2020-05-16',0), 
(8,'2020-05-17',0), 
(9,'2020-05-13',4), 
(9,'2020-05-14',2), 
(9,'2020-05-15',5), 
(9,'2020-05-16',4), 
(9,'2020-05-17',4), 
(10,'2020-05-13',2), 
(10,'2020-05-14',4), 
(10,'2020-05-15',3), 
(10,'2020-05-16',4), 
(10,'2020-05-17',4);
```
