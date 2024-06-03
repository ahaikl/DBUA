---
author: [Yuan Zhou, Weiwei Gong, Vinita Subramanian, Teck Hua Lee, Tirthankar Lahiri, Shasank Chavan, Sebastian DeLaHoz, Roger Ford, Rohan Aggarwal, Mark Hornick, Malavika S P, Harichandan Roy, George Krupka, Doug Hood, Dinesh Das, David Jiang, Boriana Milenova, Bonnie Xia, Aurosish Mishra, Angela Amor, Agnivo Saha, Aleksandra Czarlinska, Ramya P, Usha Krishnamurthy, Tulika Das, Suresh Rajan, Sarika Surampudi, Sarah Hirschfeld, Prakash Jashnani, Jody Glover, Jessica True, Mamata Basapur, Maitreyee Chaliha, Gunjan Jain, Frederick Kush, Douglas Williams, Binika Kumar, Jean-Francois Verrier]
publisherinformation: May2024
---

# Why Use Oracle AI Vector Search?

One of the biggest benefits of Oracle AI Vector Search is that semantic search on unstructured data can be combined with relational search on business data in one single system.

This is not only powerful but also significantly more effective because you don't need to add a specialized vector database, eliminating the pain of data fragmentation between multiple systems.

For example, suppose you use an application that allows you to find a house that is similar to a picture you took of one you like that is located in your preferred area for a certain budget. Finding a good match in this case requires combining a semantic picture search with searches on business data.

With Oracle AI Vector Search, you can create the following table:

```
CREATE TABLE house_for_sale (house_id     NUMBER,
                             price        NUMBER,
                             city         VARCHAR2(400),
                             house_photo  BLOB,
                             house_vector VECTOR);
```

The following sections of this guide describe in detail the meaning of the `VECTOR` data type and how to load data in this column data type.

With that table, you can run the following query to answer your basic question:

```
SELECT house_photo, city, price
FROM   house_for_sale
WHERE  price <= :input_price AND
       city  = :input_city
ORDER BY VECTOR_DISTANCE(house_vector, :input_vector);
```

Later sections of this guide describe in detail the meaning of the `VECTOR_DISTANCE` function. This query is just to show you how simple it is to combine a vector embedding similarity search with relation predicates.

**Parent topic:**[Overview](GUID-107FCC1B-EF68-4966-83CE-B7C4244E47A2.md)

