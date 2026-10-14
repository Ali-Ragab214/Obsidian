the index : طريقه لتسريع عمليه ال retrievement 
مثلا جاي اجيب row  واحد زي دا كدا 
![[Pasted image 20261012153610.png]]

اتفاجئ ان 30000 صف اتقرا وانا عايز صف واحد بس  ف لو حطيت index  عال row   دا مش هضطر اني اقرا الصفوف اللي قبله كلها => فال index  هيخلي الدنيا اسرع بكتير  وبدل ما الوقت الماخوذ يبقا مثلا 10 ثواني هيبقا 1 مللي ثانيه 

```sql 
select * from users 
where salary > 30000
```
اي اللي هيحصل لما ننفذ الكويري دي = >  ==🔵data base engine read== ==🔴pages== ==🔵from hard disk==

بص احنا عندنا الفايل اللي بيبقي فيه الداتا بتاعه الداتا بيز بيبقا mdfile  الداتا بيز مش بتقرا rows  بتقرا pages  ودي اللي بيتكون منها ال mdfile ف  كل page حجمها 8kb وعدد ال pages بيختلف علي حسب الداتا دا في ال sql  ,,,,,,, وال pages is not ordered
ال page بتتكون من 3 اجزاء 
1-Header : contain page_id
2-Data 
3-Off-set : داخلي داخل ال page 

*Full table scan* => read all pages
*Index* :Order pages based on specific column 
*Heap table* : table without a cluster index 
*Types of Indexes* : 
    1-Cluster(Most important ,Unique or Not) : this one will order your table 
    2-Non Cluster(Unique or not) : 

1-> Cluster 
اعرف ال Binary search tree الاول...بص هو الاندكس مش شغال بيها بس نعرفها لان ال tree  هتسرع العمليه جدا بس ال index is not  Binary search tree

هندخل علي حاجه اسمها  => Balance Tree 
تختلف عن ال bst ان ال node ممكن تحتوي علي اكثر من key تاني حاجه فيه  من 2 اطفال ليها يس 

لو عندي 3 child يبقا ال root node فيها 2 
![[Pasted image 20261013145955.png]] 
 ![[Pasted image 20261013145955.png]]

![[Pasted image 20261013145542.png]]
Non-Cluster عشان كدا مش مترتب


احنا هنخلي الداتا في ال leaf node    بس ودا اسمه B+ tree  وال leaf nodes دي مرتبطين مع بعض ك doubly linked list
![[Pasted image 20261013150714.png]]

ودي فيها نوعين من ال pages :
1-indexing(navigation) pages
2-Data pages
ودا cluster index
ودي المقدمه