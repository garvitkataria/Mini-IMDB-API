-- Please Open In SUBILME TEXT EDITOR
Garvit Kataria 201601028

1. 4 queries with atleast 7 joins in each.
2. For each query, proper analysis with indexes and without indexes is done.
3. Total of 2 Procedures with IN and OUT variables have same queries to compare performance.

General Approach:
1. Query Performance is reported with possible foreign keys and indexing
2.All indexes are removed and query performance is calculated and reported.

General Assumptions:
1. Mysql automatically creates BTREE index on primary key and foreign key and hence indexes on same field are not made.
2. These indexes can be dropped once foreign key constraint is dropped.
3. Indexing on primary key is not dropped because of permission requirements and other constraints.
4. Hence, anywhere in the following document any kind of indexing on primary key is not mentioned.
5. Mysql automatically creates SPATIAL index on geometric data and hence indexes on same field are not made.
6. We are using InnoDB Engines in mysql. InnoDB engine silently changes a hash index to a BTREE index. Hence, no hash index could possibly be shown.

General Observations Needed From Query Optimization:
1. In A join B using, if we use A.a = B.b, then foreign key indexing of A matters but if we use B.b = A.a then primary key indexing of B matters which we made an assumption that will not be removed. So order of join matters a lot



SELECT  PlaylistTrack.PlaylistId,  Playlist_name, TrackName, Composer, AlbumTitle, ArtistName, MediaTypeName, GenreName, Milliseconds, Bytes, TrackPrice 
FROM PlaylistTrack 
INNER JOIN Playlist 
ON PlaylistTrack.PlaylistId = Playlist.PlaylistId 
INNER JOIN (
	SELECT * FROM Track 
	INNER JOIN 
	(SELECT * FROM Album 
		INNER JOIN Artist 
		ON Album.Artist_Id = Artist.ArtistId
	) z 
	ON Track.AlbumId = z.Album_Id 
	INNER JOIN MediaType 
	ON Track.MediaTypeId = MediaType.MediaType_Id 
	INNER JOIN Genre 
	ON Track.GenreId = Genre.Genre_Id) p 
ON PlaylistTrack.TrackId = p.Track_Id;


+----+-------------+---------------+------------+--------+----------------------------------------------------------------+--------------------------+---------+----------------------------------+------+----------+---------------------------------------+
| id | select_type | table         | partitions | type   | possible_keys                                                  | key                      | key_len | ref                              | rows | filtered | Extra                                 |
+----+-------------+---------------+------------+--------+----------------------------------------------------------------+--------------------------+---------+----------------------------------+------+----------+---------------------------------------+
|  1 | SIMPLE      | MediaType     | NULL       | ALL    | PRIMARY                                                        | NULL                     | NULL    | NULL                             |    5 |   100.00 | NULL                                  |
|  1 | SIMPLE      | Genre         | NULL       | ALL    | PRIMARY                                                        | NULL                     | NULL    | NULL                             |   25 |   100.00 | Using join buffer (Block Nested Loop) |
|  1 | SIMPLE      | Track         | NULL       | ref    | PRIMARY,IFK_TrackAlbumId,IFK_TrackGenreId,IFK_TrackMediaTypeId | IFK_TrackGenreId         | 5       | chinook.Genre.Genre_Id           |  139 |    20.00 | Using where                           |
|  1 | SIMPLE      | Album         | NULL       | eq_ref | PRIMARY,IFK_AlbumArtistId                                      | PRIMARY                  | 4       | chinook.Track.AlbumId            |    1 |   100.00 | Using where                           |
|  1 | SIMPLE      | Artist        | NULL       | eq_ref | PRIMARY                                                        | PRIMARY                  | 4       | chinook.Album.Artist_Id          |    1 |   100.00 | NULL                                  |
|  1 | SIMPLE      | PlaylistTrack | NULL       | ref    | PRIMARY,IFK_PlaylistTrackTrackId                               | IFK_PlaylistTrackTrackId | 4       | chinook.Track.Track_id           |    2 |   100.00 | Using index                           |
|  1 | SIMPLE      | Playlist      | NULL       | eq_ref | PRIMARY                                                        | PRIMARY                  | 4       | chinook.PlaylistTrack.PlaylistId |    1 |   100.00 | NULL                                  |
+----+-------------+---------------+------------+--------+----------------------------------------------------------------+--------------------------+---------+----------------------------------+------+----------+---------------------------------------+
Last_query_cost = 22862.134293


+----+-------------+---------------+------------+--------+---------------+---------+---------+----------------------------------------------------+------+----------+----------------------------------------------------+
| id | select_type | table         | partitions | type   | possible_keys | key     | key_len | ref                                                | rows | filtered | Extra                                              |
+----+-------------+---------------+------------+--------+---------------+---------+---------+----------------------------------------------------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | MediaType     | NULL       | ALL    | PRIMARY       | NULL    | NULL    | NULL                                               |    5 |   100.00 | NULL                                               |
|  1 | SIMPLE      | Track         | NULL       | ALL    | PRIMARY       | NULL    | NULL    | NULL                                               | 3483 |    10.00 | Using where; Using join buffer (Block Nested Loop) |
|  1 | SIMPLE      | Genre         | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | chinook.Track.GenreId                              |    1 |   100.00 | NULL                                               |
|  1 | SIMPLE      | Album         | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | chinook.Track.AlbumId                              |    1 |   100.00 | Using where                                        |
|  1 | SIMPLE      | Artist        | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | chinook.Album.Artist_Id                            |    1 |   100.00 | NULL                                               |
|  1 | SIMPLE      | Playlist      | NULL       | ALL    | PRIMARY       | NULL    | NULL    | NULL                                               |   18 |   100.00 | Using join buffer (Block Nested Loop)              |
|  1 | SIMPLE      | PlaylistTrack | NULL       | eq_ref | PRIMARY       | PRIMARY | 8       | chinook.Playlist.PlaylistId,chinook.Track.Track_id |    1 |   100.00 | Using index                                        |
+----+-------------+---------------+------------+--------+---------------+---------+---------+----------------------------------------------------+------+----------+----------------------------------------------------+
Last_query_cost = 24414.177858 


SELECT * 
FROM(
	SELECT * 
	FROM InvoiceLine 
	INNER JOIN Invoice 
	ON InvoiceLine.InvoiceId = Invoice.Invoice_Id 
	INNER JOIN (
		SELECT * 
		FROM Track 
		INNER JOIN (
			SELECT * 
			FROM Album 
			INNER JOIN Artist 
			ON Album.Artist_Id = Artist.ArtistId
			) z 
		ON Track.AlbumId = z.Album_Id 
		INNER JOIN MediaType 
		ON Track.MediaTypeId = MediaType.MediaType_Id 
		INNER JOIN Genre 
		ON Track.GenreId = Genre.Genre_Id
		)k 
	ON InvoiceLine.TrackId = k.Track_Id)y 
INNER JOIN Customer 
ON y.CustomerId=Customer.Customer_Id;


+----+-------------+-------------+------------+--------+----------------------------------------------------------------+--------------------------+---------+------------------------------+------+----------+---------------------------------------+
| id | select_type | table       | partitions | type   | possible_keys                                                  | key                      | key_len | ref                          | rows | filtered | Extra                                 |
+----+-------------+-------------+------------+--------+----------------------------------------------------------------+--------------------------+---------+------------------------------+------+----------+---------------------------------------+
|  1 | SIMPLE      | Customer    | NULL       | ALL    | PRIMARY                                                        | NULL                     | NULL    | NULL                         |   59 |   100.00 | NULL                                  |
|  1 | SIMPLE      | Invoice     | NULL       | ref    | PRIMARY,IFK_InvoiceCustomerId                                  | IFK_InvoiceCustomerId    | 4       | chinook.Customer.Customer_Id |    6 |   100.00 | NULL                                  |
|  1 | SIMPLE      | InvoiceLine | NULL       | ref    | IFK_InvoiceLineInvoiceId,IFK_InvoiceLineTrackId                | IFK_InvoiceLineInvoiceId | 4       | chinook.Invoice.invoice_id   |    5 |   100.00 | NULL                                  |
|  1 | SIMPLE      | MediaType   | NULL       | ALL    | PRIMARY                                                        | NULL                     | NULL    | NULL                         |    5 |   100.00 | Using join buffer (Block Nested Loop) |
|  1 | SIMPLE      | Track       | NULL       | eq_ref | PRIMARY,IFK_TrackAlbumId,IFK_TrackGenreId,IFK_TrackMediaTypeId | PRIMARY                  | 4       | chinook.InvoiceLine.TrackId  |    1 |    20.00 | Using where                           |
|  1 | SIMPLE      | Genre       | NULL       | eq_ref | PRIMARY                                                        | PRIMARY                  | 4       | chinook.Track.GenreId        |    1 |   100.00 | NULL                                  |
|  1 | SIMPLE      | Album       | NULL       | eq_ref | PRIMARY,IFK_AlbumArtistId                                      | PRIMARY                  | 4       | chinook.Track.AlbumId        |    1 |   100.00 | Using where                           |
|  1 | SIMPLE      | Artist      | NULL       | eq_ref | PRIMARY                                                        | PRIMARY                  | 4       | chinook.Album.Artist_Id      |    1 |   100.00 | NULL                                  |
+----+-------------+-------------+------------+--------+----------------------------------------------------------------+--------------------------+---------+------------------------------+------+----------+---------------------------------------+
Last_query_cost = 12621.373604 


+----+-------------+-------------+------------+--------+---------------+---------+---------+-------------------------------+------+----------+---------------------------------------+
| id | select_type | table       | partitions | type   | possible_keys | key     | key_len | ref                           | rows | filtered | Extra                                 |
+----+-------------+-------------+------------+--------+---------------+---------+---------+-------------------------------+------+----------+---------------------------------------+
|  1 | SIMPLE      | MediaType   | NULL       | ALL    | PRIMARY       | NULL    | NULL    | NULL                          |    5 |   100.00 | NULL                                  |
|  1 | SIMPLE      | InvoiceLine | NULL       | ALL    | NULL          | NULL    | NULL    | NULL                          | 2240 |   100.00 | Using join buffer (Block Nested Loop) |
|  1 | SIMPLE      | Invoice     | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | chinook.InvoiceLine.InvoiceId |    1 |   100.00 | NULL                                  |
|  1 | SIMPLE      | Customer    | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | chinook.Invoice.CustomerId    |    1 |   100.00 | NULL                                  |
|  1 | SIMPLE      | Track       | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | chinook.InvoiceLine.TrackId   |    1 |    10.00 | Using where                           |
|  1 | SIMPLE      | Genre       | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | chinook.Track.GenreId         |    1 |   100.00 | NULL                                  |
|  1 | SIMPLE      | Album       | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | chinook.Track.AlbumId         |    1 |   100.00 | Using where                           |
|  1 | SIMPLE      | Artist      | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | chinook.Album.Artist_Id       |    1 |   100.00 | NULL                                  |
+----+-------------+-------------+------------+--------+---------------+---------+---------+-------------------------------+------+----------+---------------------------------------+
Last_query_cost = 17034.015194


___________________________________________________________________________________________________________________________________
DELIMITER $$
DROP PROCEDURE IF EXISTS TracksByCustomer$$
CREATE PROCEDURE TracksByCustomer(CId INT, OUT NoOfTracks FLOAT, OUT Bill FLOAT)
BEGIN
    select count(InvoiceId) into NoOfTracks 
    FROM(
    	SELECT * 
    	FROM InvoiceLine 
    	INNER JOIN Invoice 
    	ON InvoiceLine.InvoiceId = Invoice.Invoice_Id 
    	INNER JOIN (
    		SELECT * 
    		FROM Track 
    		INNER JOIN (
    			SELECT * 
    			FROM Album 
    			INNER JOIN Artist 
    			ON Album.Artist_Id = Artist.ArtistId
    			) z ON Track.AlbumId = z.Album_Id 
    		INNER JOIN MediaType 
    		ON Track.MediaTypeId = MediaType.MediaType_Id 
    		INNER JOIN Genre 
    		ON Track.GenreId = Genre.Genre_Id)k 
    	ON InvoiceLine.TrackId = k.Track_Id)y 
    INNER JOIN Customer 
    ON y.CustomerId=Customer.Customer_Id 
    where Customer_Id=CId;
    select SUM(UnitPrice) into Bill FROM(SELECT * FROM InvoiceLine INNER JOIN Invoice ON InvoiceLine.InvoiceId = Invoice.Invoice_Id INNER JOIN (SELECT * FROM Track INNER JOIN (SELECT * FROM Album INNER JOIN Artist ON Album.Artist_Id = Artist.ArtistId) z ON Track.AlbumId = z.Album_Id INNER JOIN MediaType ON Track.MediaTypeId = MediaType.MediaType_Id INNER JOIN Genre ON Track.GenreId = Genre.Genre_Id)k ON InvoiceLine.TrackId = k.Track_Id)y INNER JOIN Customer ON y.CustomerId=Customer.Customer_Id where Customer_Id=CId;
END$$
DELIMITER ;
Call TracksByCustomer(1,@NoOfTracks,@Bill);
Select @NoOfTracks;
Select @Bill;


DELIMITER $$
DROP PROCEDURE IF EXISTS TracksSold$$
CREATE PROCEDURE TracksSold(Tid INT, OUT NoOfCopies FLOAT)
BEGIN
    Select count(TrackId) into NoOfCopies FROM(SELECT * FROM InvoiceLine INNER JOIN Invoice ON InvoiceLine.InvoiceId = Invoice.Invoice_Id INNER JOIN (SELECT * FROM Track INNER JOIN (SELECT * FROM Album INNER JOIN Artist ON Album.Artist_Id = Artist.ArtistId) z ON Track.AlbumId = z.Album_Id INNER JOIN MediaType ON Track.MediaTypeId = MediaType.MediaType_Id INNER JOIN Genre ON Track.GenreId = Genre.Genre_Id)k ON InvoiceLine.TrackId = k.Track_Id)y INNER JOIN Customer ON y.CustomerId=Customer.Customer_Id where TrackId=Tid ;
END$$
DELIMITER ;
CALL TracksSold(1,@NoOfCopies);
Select @NoOfCopies;
___________________________________________________________________________________________________________________________________

=> Analysis Of queries used in PROCEDURES
select CustomerId, count(InvoiceId) as NoOfTracks, SUM(UnitPrice) as Bill 
FROM(
	SELECT * 
	FROM InvoiceLine 
	INNER JOIN Invoice 
	ON InvoiceLine.InvoiceId = Invoice.Invoice_Id 
	INNER JOIN (
		SELECT * 
		FROM Track 
		INNER JOIN (
			SELECT * 
			FROM Album 
			INNER JOIN Artist 
			ON Album.Artist_Id = Artist.ArtistId
			) z 
		ON Track.AlbumId = z.Album_Id 
		INNER JOIN MediaType 
		ON Track.MediaTypeId = MediaType.MediaType_Id 
		INNER JOIN Genre 
		ON Track.GenreId = Genre.Genre_Id
		)k 
	ON InvoiceLine.TrackId = k.Track_Id
	)y 
INNER JOIN Customer 
ON y.CustomerId=Customer.Customer_Id 
GROUP BY Customer_Id 
ORDER BY Bill DESC;

+----+-------------+-------------+------------+--------+----------------------------------------------------------------+--------------------------+---------+-----------------------------------+------+----------+----------------------------------------------------+
| id | select_type | table       | partitions | type   | possible_keys                                                  | key                      | key_len | ref                               | rows | filtered | Extra                                              |
+----+-------------+-------------+------------+--------+----------------------------------------------------------------+--------------------------+---------+-----------------------------------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | Customer    | NULL       | index  | PRIMARY,IFK_CustomerSupportRepId                               | IFK_CustomerSupportRepId | 5       | NULL                              |   59 |   100.00 | Using index; Using temporary; Using filesort       |
|  1 | SIMPLE      | Invoice     | NULL       | ref    | PRIMARY,IFK_InvoiceCustomerId                                  | IFK_InvoiceCustomerId    | 4       | chinookindex.Customer.Customer_Id |    6 |   100.00 | Using index                                        |
|  1 | SIMPLE      | InvoiceLine | NULL       | ref    | IFK_InvoiceLineInvoiceId,IFK_InvoiceLineTrackId                | IFK_InvoiceLineInvoiceId | 4       | chinookindex.Invoice.Invoice_Id   |    5 |   100.00 | NULL                                               |
|  1 | SIMPLE      | MediaType   | NULL       | index  | PRIMARY                                                        | PRIMARY                  | 4       | NULL                              |    5 |   100.00 | Using index; Using join buffer (Block Nested Loop) |
|  1 | SIMPLE      | Track       | NULL       | eq_ref | PRIMARY,IFK_TrackAlbumId,IFK_TrackGenreId,IFK_TrackMediaTypeId | PRIMARY                  | 4       | chinookindex.InvoiceLine.TrackId  |    1 |    20.00 | Using where                                        |
|  1 | SIMPLE      | Genre       | NULL       | eq_ref | PRIMARY                                                        | PRIMARY                  | 4       | chinookindex.Track.GenreId        |    1 |   100.00 | Using index                                        |
|  1 | SIMPLE      | Album       | NULL       | eq_ref | PRIMARY,IFK_AlbumArtistId                                      | PRIMARY                  | 4       | chinookindex.Track.AlbumId        |    1 |   100.00 | Using where                                        |
|  1 | SIMPLE      | Artist      | NULL       | eq_ref | PRIMARY                                                        | PRIMARY                  | 4       | chinookindex.Album.Artist_Id      |    1 |   100.00 | Using index                                        |
+----+-------------+-------------+------------+--------+----------------------------------------------------------------+--------------------------+---------+-----------------------------------+------+----------+----------------------------------------------------+
Last_query_cost = 12251.756327 


+----+-------------+-------------+------------+--------+----------------------------------+---------+---------+-------------------------------+------+----------+----------------------------------------------+
| id | select_type | table       | partitions | type   | possible_keys                    | key     | key_len | ref                           | rows | filtered | Extra                                        |
+----+-------------+-------------+------------+--------+----------------------------------+---------+---------+-------------------------------+------+----------+----------------------------------------------+
|  1 | SIMPLE      | MediaType   | NULL       | index  | PRIMARY                          | PRIMARY | 4       | NULL                          |    5 |   100.00 | Using index; Using temporary; Using filesort |
|  1 | SIMPLE      | InvoiceLine | NULL       | ALL    | NULL                             | NULL    | NULL    | NULL                          | 2240 |   100.00 | Using join buffer (Block Nested Loop)        |
|  1 | SIMPLE      | Invoice     | NULL       | eq_ref | PRIMARY                          | PRIMARY | 4       | chinook.InvoiceLine.InvoiceId |    1 |   100.00 | NULL                                         |
|  1 | SIMPLE      | Customer    | NULL       | eq_ref | PRIMARY,IFK_CustomerSupportRepId | PRIMARY | 4       | chinook.Invoice.CustomerId    |    1 |   100.00 | Using index                                  |
|  1 | SIMPLE      | Track       | NULL       | eq_ref | PRIMARY                          | PRIMARY | 4       | chinook.InvoiceLine.TrackId   |    1 |    10.00 | Using where                                  |
|  1 | SIMPLE      | Genre       | NULL       | eq_ref | PRIMARY                          | PRIMARY | 4       | chinook.Track.GenreId         |    1 |   100.00 | Using index                                  |
|  1 | SIMPLE      | Album       | NULL       | eq_ref | PRIMARY                          | PRIMARY | 4       | chinook.Track.AlbumId         |    1 |   100.00 | Using where                                  |
|  1 | SIMPLE      | Artist      | NULL       | eq_ref | PRIMARY                          | PRIMARY | 4       | chinook.Album.Artist_Id       |    1 |   100.00 | Using index                                  |
+----+-------------+-------------+------------+--------+----------------------------------+---------+---------+-------------------------------+------+----------+----------------------------------------------+
Last_query_cost = 17033.999630 


select TrackId,TrackName,ArtistName,AlbumTitle,Composer,TrackPrice,MediaTypeName,GenreName, count(TrackId) as SoldCopies 
FROM(
	SELECT * 
	FROM InvoiceLine 
	INNER JOIN Invoice 
	ON InvoiceLine.InvoiceId = Invoice.Invoice_Id 
	INNER JOIN (
		SELECT * 
		FROM Track 
		INNER JOIN (
			SELECT * 
			FROM Album 
			INNER JOIN Artist 
			ON Album.Artist_Id = Artist.ArtistId
			) z 
		ON Track.AlbumId = z.Album_Id 
		INNER JOIN MediaType 
		ON Track.MediaTypeId = MediaType.MediaType_Id 
		INNER JOIN Genre 
		ON Track.GenreId = Genre.Genre_Id
		)k 
	ON InvoiceLine.TrackId = k.Track_Id
	)y 
INNER JOIN Customer 
ON y.CustomerId=Customer.Customer_Id 
GROUP BY TrackId 
ORDER BY SoldCopies DESC;
+----+-------------+-------------+------------+--------+----------------------------------------------------------------+--------------------------+---------+-----------------------------------+------+----------+----------------------------------------------+
| id | select_type | table       | partitions | type   | possible_keys                                                  | key                      | key_len | ref                               | rows | filtered | Extra                                        |
+----+-------------+-------------+------------+--------+----------------------------------------------------------------+--------------------------+---------+-----------------------------------+------+----------+----------------------------------------------+
|  1 | SIMPLE      | Customer    | NULL       | index  | PRIMARY                                                        | IFK_CustomerSupportRepId | 5       | NULL                              |   59 |   100.00 | Using index; Using temporary; Using filesort |
|  1 | SIMPLE      | Invoice     | NULL       | ref    | PRIMARY,IFK_InvoiceCustomerId                                  | IFK_InvoiceCustomerId    | 4       | chinookindex.Customer.Customer_Id |    6 |   100.00 | Using index                                  |
|  1 | SIMPLE      | InvoiceLine | NULL       | ref    | IFK_InvoiceLineInvoiceId,IFK_InvoiceLineTrackId                | IFK_InvoiceLineInvoiceId | 4       | chinookindex.Invoice.Invoice_Id   |    5 |   100.00 | NULL                                         |
|  1 | SIMPLE      | MediaType   | NULL       | ALL    | PRIMARY                                                        | NULL                     | NULL    | NULL                              |    5 |   100.00 | Using join buffer (Block Nested Loop)        |
|  1 | SIMPLE      | Track       | NULL       | eq_ref | PRIMARY,IFK_TrackAlbumId,IFK_TrackGenreId,IFK_TrackMediaTypeId | PRIMARY                  | 4       | chinookindex.InvoiceLine.TrackId  |    1 |    20.00 | Using where                                  |
|  1 | SIMPLE      | Genre       | NULL       | eq_ref | PRIMARY                                                        | PRIMARY                  | 4       | chinookindex.Track.GenreId        |    1 |   100.00 | NULL                                         |
|  1 | SIMPLE      | Album       | NULL       | eq_ref | PRIMARY,IFK_AlbumArtistId                                      | PRIMARY                  | 4       | chinookindex.Track.AlbumId        |    1 |   100.00 | Using where                                  |
|  1 | SIMPLE      | Artist      | NULL       | eq_ref | PRIMARY                                                        | PRIMARY                  | 4       | chinookindex.Album.Artist_Id      |    1 |   100.00 | NULL                                         |
+----+-------------+-------------+------------+--------+----------------------------------------------------------------+--------------------------+---------+-----------------------------------+------+----------+----------------------------------------------+

Last_query_cost = 14251.713602
+----+-------------+-------------+------------+--------+---------------+---------+---------+-------------------------------+------+----------+---------------------------------------+
| id | select_type | table       | partitions | type   | possible_keys | key     | key_len | ref                           | rows | filtered | Extra                                 |
+----+-------------+-------------+------------+--------+---------------+---------+---------+-------------------------------+------+----------+---------------------------------------+
|  1 | SIMPLE      | MediaType   | NULL       | ALL    | PRIMARY       | NULL    | NULL    | NULL                          |    5 |   100.00 | Using temporary; Using filesort       |
|  1 | SIMPLE      | InvoiceLine | NULL       | ALL    | NULL          | NULL    | NULL    | NULL                          | 2240 |   100.00 | Using join buffer (Block Nested Loop) |
|  1 | SIMPLE      | Invoice     | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | chinook.InvoiceLine.InvoiceId |    1 |   100.00 | NULL                                  |
|  1 | SIMPLE      | Customer    | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | chinook.Invoice.CustomerId    |    1 |   100.00 | Using index                           |
|  1 | SIMPLE      | Track       | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | chinook.InvoiceLine.TrackId   |    1 |    10.00 | Using where                           |
|  1 | SIMPLE      | Genre       | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | chinook.Track.GenreId         |    1 |   100.00 | NULL                                  |
|  1 | SIMPLE      | Album       | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | chinook.Track.AlbumId         |    1 |   100.00 | Using where                           |
|  1 | SIMPLE      | Artist      | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | chinook.Album.Artist_Id       |    1 |   100.00 | NULL                                  |
+----+-------------+-------------+------------+--------+---------------+---------+---------+-------------------------------+------+----------+---------------------------------------+
Last_query_cost = 19034.015194
___________________________________________________________________________________________________________________________________

ADDING INDEXES OF OUR OWN - 

CASE 1 -  
	INDEXED AlbumTitle Attribute of Table - album
	mysql> SHOW INDEX FROM album;
	+-------+------------+------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
	| Table | Non_unique | Key_name         | Seq_in_index | Column_name | Collation| Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment
	 |
	+-------+------------+------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
	| album |          0 | PRIMARY          |            1 | Album_Id    | A        |         347 |     NULL | NULL   |      | BTREE      |         |
	| album |          1 | album_name_index |            1 | AlbumTitle  | A        |         346 |     NULL | NULL   | YES  | BTREE      |         |
	|
	+-------+------------+------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+

	mysql> SELECT * from album WHERE AlbumTitle='Tribute';
	+----------+------------+-----------+
	| Album_Id | AlbumTitle | Artist_Id |
	+----------+------------+-----------+
	|      174 | Tribute    |       114 |
	+----------+------------+-----------+
	1 row in set (0.00 sec)

	mysql> SHOW status like 'last_query_cost';
	+-----------------+----------+
	| Variable_name   | Value    |
	+-----------------+----------+
	| Last_query_cost | 1.199000 |
	+-----------------+----------+
	1 row in set (0.00 sec)

	When No index-

	mysql> SELECT * FROM album WHERE AlbumTitle='Tribute';
	+----------+------------+-----------+
	| Album_Id | AlbumTitle | Artist_Id |
	+----------+------------+-----------+
	|      174 | Tribute    |       114 |
	+----------+------------+-----------+
	1 row in set (0.01 sec)

	mysql> SHOW status like 'last_query_cost';
	+-----------------+-----------+
	| Variable_name   | Value     |
	+-----------------+-----------+
	| Last_query_cost | 72.399000 |
	+-----------------+-----------+
	1 row in set (0.00 sec)

	OBSERVATION- Clearly Cost is coming drastically less in an indexed table when searched by the indexed attribute.
 
 CASE 2

 	mysql> SHOW INDEX FROM Customer;
	+----------+------------+--------------------------+--------------+--------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
	| Table    | Non_unique | Key_name                 | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
	+----------+------------+--------------------------+--------------+--------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
	| customer |          0 | PRIMARY                  |            1 | Customer_Id | A         |          59 |     NULL | NULL   |      | BTREE      |         |               |
	| customer |          1 | IFK_CustomerSupportRepId |            1 | SupportRepId| A         |           3 |     NULL | NULL   | YES  | BTREE      |         |               |
	| customer |          1 | name                     |            1 | FirstName   | A         |          56 |     NULL | NULL   |      | BTREE      |         |               |
	+----------+------------+--------------------------+--------------+--------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
	3 rows in set (0.00 sec)

	SELECT * 
	FROM(
		SELECT * 
		FROM InvoiceLine 
		INNER JOIN Invoice 
		ON InvoiceLine.InvoiceId = Invoice.Invoice_Id 
		INNER JOIN (
			SELECT * 
			FROM Track 
			INNER JOIN (
				SELECT * 
				FROM Album 
				INNER JOIN Artist 
				ON Album.Artist_Id = Artist.ArtistId
				) z 
			ON Track.AlbumId = z.Album_Id 
			INNER JOIN MediaType 
			ON Track.MediaTypeId = MediaType.MediaType_Id 
			INNER JOIN Genre 
			ON Track.GenreId = Genre.Genre_Id
			)k 
		ON InvoiceLine.TrackId = k.Track_Id
		)y 
		INNER JOIN Customer ON y.CustomerId=Customer.Customer_Id 
		WHERE firstname = "Manoj";


	mysql> SHOW status like 'last_query_cost';
	+-----------------+-------------+
	| Variable_name   | Value       |
	+-----------------+-------------+
	| Last_query_cost | 3954.358200 |
	+-----------------+-------------+
	1 row in set (0.00 sec)


	When No Index-

	mysql> SHOW status like 'last_query_cost';
	+-----------------+-------------+
	| Variable_name   | Value       |
	+-----------------+-------------+
	| Last_query_cost | 6641.116543 |
	+-----------------+-------------+
	1 row in set (0.00 sec)


	OBSERVATION - The cost is coming as almost half of what we get when we have the 'firstName' attribute indexed.




