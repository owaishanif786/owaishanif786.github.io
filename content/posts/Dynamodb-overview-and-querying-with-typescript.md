---
title: "Dynamodb Overview and Querying With Typescript"
date: 2022-08-08T04:57:00+05:00
draft: false
toc: true
featured_image: '/images/dynamo-typescript.png'

---
![Image alt](/images/dynamo-typescript.png)



# Dynamodb Overview and Querying with Typescript/Node

Two main things while defining table.
1. Partition Key
2. Sort Key

For simplicity I have selected `On Demand` Capcity mode. 

## Partition Key or Hash Key
Partition key is the must thing you need to define while creating table. This is the default index based on which dynamo indexes entire table data.

Keys suited for partition Key:
- Anything that can be unique i.e IDs, file_paths etc
- If you don't have unique partition key than you can also use sort key along side partition key. that will become compound key. However that compound key must be unique. 

## Sort key or Range Key
This is optional but you can use this further to sort data or while querying it can help you filter data quickly. 

Best keys suited for sort key:
- Organize and Combine multiple keys into sort key such that you can easily query with begins_with and <>. for example [country]#[region]#[state]#[county]#[city]#[neighborhood]. 
Notice country is big namespace where everything can live in it. 
- Timestamps can also be a good option for sort key


```
hash key = partition key
range key = sort key
```


It is not necessary that partition key should always be unique. `status` fields are good candidate for partition key but you have to choose something unique sort key so you can filter easily. for example let say we have to track files which are being copied from one place to another. we can choose `file_status` as partition key and `file_path` as sort key. then we will be able to query very easily.
To check all files of folderX whose file_status is inProgress. 

```sql
select * from files where  file_status = 'inProgress' and file_path begins_with 'folder1/folder2/folderX' 
```
Normally we won't use above query syntax in dynamodb but there are some wrappers like in Bit-Redash which allows these kind of queries. Notice the begins_with operator can only be used with sort keys. There are other operators you can use with sort keys `= < > <= >= between and begins_with`


### Local Secondary Index
- same partition key as the table have but with different sort/range key.
- can only be created at the time of table creation
- LSI supports both eventual and strong consistent read.

### Global Secondary Index
- we choose different partition key from the table default partition key however sort key can be any column.
- Can be created after table creation
- GSI only supports eventual consistency.



#### Example: Books Table Indexes

Default Index 

|Partition key                |Sort key                          
|----------------|-------------------------------|
|ISBN (String)|published_date (String)           |  



 ## Global secondary indexes 

|Name                |Partition Key                          |Sort Key                         |
|----------------|-------------------------------|-----------------------------|
|   -published_date-index |category (String)            |published_date (String)  



## Local Secondary Indexes

|Name                |Partition Key                          |Sort Key                         |Projected Attributes    |
|----------------|-------------------------------|-----------------------------|------------|
|author-index	|ISBN (String)	            |author (String)	  |All
|category-index	|ISBN (String)	|category (String)	|All


### Example Table Data
|ISBN        |published_date          |author|category|title                    |
|------------|------------------------|------|--------|-------------------------|
|333333333   |2022-07-10T20:21:54.094Z|jane  |earth   |everyhing green          |
|222222222   |2022-07-24T20:21:54.094Z|billy |earth   |plants and their kingdoms|
|111111111   |2022-08-07T20:20:10.528Z|joe   |space   |seven sky                |
|RJxWIQOvoZUC|2022-08-07T20:17:11.763Z|Gail  |fun     |oceans and sky           |


```typescript
import { DynamoDBClient, QueryCommand, QueryCommandInput, QueryCommandOutput } from "@aws-sdk/client-dynamodb";
import config  from './config.js';

const client = new DynamoDBClient({ region: config.dynamoRegion });


interface IBook {
    ISBN: string;
    published_date: string;
    author:string;
    category:string;
    title:string;
}


//Querying on default Partition key and sort key. 
//Notice we are not using any parameter IndexName because we are querying on default partition key/index.
//When querying default index we have to provide partition key and if sort key is included like in this case then we must also provide sort key
//Notice ISBN is unique and published_date as sort key does not make any sense. so when we are choosing partition key as totally unique then there is no need to pick sort key.
async function getBook(ISBN:string, published_date:string ):Promise<IBook | null> {
    try {
        const input:QueryCommandInput = {
            TableName: config.TableName, 
            KeyConditionExpression: "#ISBN = :ISBN and #published_date = :published_date",
            ExpressionAttributeNames: {
                "#ISBN": "ISBN",
                "#published_date": "published_date",
              },
              ExpressionAttributeValues: {
                ":ISBN": { S: ISBN },
                ":published_date": { S: published_date },
              },
              Select:"ALL_ATTRIBUTES"
           };
        const command = new QueryCommand(input);
        const data:QueryCommandOutput = await client.send(command);
        if(data.Items?.length && data.Items[0]){
            let book = data.Items[0];
            // console.log(book)

        
            return {
                ISBN: book.ISBN.S!,
                published_date: book.published_date.S!,
                author:book.author.S!,
                category:book.category.S!,
                title:book.title.S!
            }  
        }
      } catch (error) {
        console.log('[E] Error', error)
       
        // return error;
        // error handling.
      } 
      return null;
      

}


//We are querying on GSI category as partition key and published_date as sort key
//Notice function call in run ["earth", "2022-07"] where we are asking select * from books where category = earth and published_date begins_with "2022-07"
//If we have lot more record then we can also use FilterExpress that can further narrow down. with FilterExpression we can use almost all operators we can use with sort key. like begins_with <>= etc.
async function listBook(category:string, published_date:string ):Promise<IBook[] | null> {
    try {
        const input:QueryCommandInput = {
            TableName: config.TableName, 
            IndexName: "category-published_date-index",
            KeyConditionExpression: "#category = :category and begins_with(#published_date, :published_date) ",
            ExpressionAttributeNames: {
                "#category": "category",
                "#published_date": "published_date",
              },
              ExpressionAttributeValues: {
                ":category": { S: category },
                ":published_date": { S: published_date },
              },
              Select:"ALL_ATTRIBUTES"
           };
        const command = new QueryCommand(input);
        const data:QueryCommandOutput = await client.send(command);
        if(data.Items?.length && data.Items[0]){
            let books = data.Items;
            // console.log(book)

            return books.map(book => ({
                ISBN: book.ISBN.S!,
                published_date: book.published_date.S!,
                author:book.author.S!,
                category:book.category.S!,
                title:book.title.S!
            }  ))
        
           
        }
      } catch (error) {
        console.log('[E] Error', error)
       
        // return error;
        // error handling.
      } 
      return null;
      

}




async function run():Promise<void>{
    const book:IBook | null = await getBook("111111111", "2022-08-07T20:20:10.528Z");
    console.log('[I] book:', book);

    const bookList:IBook[] | null = await listBook("earth", "2022-07");
    console.log('[I] bookList: ', bookList)

}

run();

/*
[I] book: {
  ISBN: '111111111',
  published_date: '2022-08-07T20:20:10.528Z',
  author: 'joe',
  category: 'space',
  title: 'seven sky'
}
[I] bookList:  [
  {
    ISBN: '333333333',
    published_date: '2022-07-10T20:21:54.094Z',
    author: 'jane',
    category: 'earth',
    title: 'everyhing green'
  },
  {
    ISBN: '222222222',
    published_date: '2022-07-24T20:21:54.094Z',
    author: 'billy',
    category: 'earth',
    title: 'plants and their kingdoms'
  }
]
*/

```



