# Generals

Download qq8e from [here](https://github.com/qq8e/qq)

This database actually contains 719753240 relations , but 51278021 have clash with others . 😔

So I've got 668354125 people's phone number , but some of them has 2 or more phone number . 🤬

Anyway , it is a big social engineering database . Storing it with normal database software wastes a lot of disk space .

Here I'd like to introduce a better way to manage the database using less disk space .

# Preprocessing and Encode

The origin file is a text , each line comes "x----z" or "x----y----z" . The last "z" means phone number , numbers in front of it means qqid .

Scan it line by line and split numbers by "----" , each line we have several relations .

QQid ranges `[0,0xffffffff]` , storaging it by `unsigned int` takes 4 bytes each .

Phone number ranges `[130-0000-0000,199-9999-9999]` in Chinese , but not every prefix is taken .

Use bucket to check how many prefix is in use , in all 10000 `five digits prefix which leads by 1` , it takes 4100 .

Now we can map every number to an `unsigned int` , it takes 4bytes each also .

Then encode all relations like `4 bytes QQid , 4 bytes phone number` , every struct takes 8 bytes .

Finally , it takes about `5.36GiB` after encoding .

# Sort

To search the key quickly , we have to `sort` the data for the next `dichotomous` . But if your computer has `<8GB` memory , it is difficult to load the file.

But we can use disk as an extra memory . We can split file into many parts , sort them separately . After sorting every part , each time we pick the minimum key and move it to the tail of final sequence .

During the process , twice size of data was read from and wrote to disk .

# Query

Now our data has been sorted , we can `dichotomous` it conveniently . But the file is still too large .

We can discover that the data range is `4.2*10^9` , the number of relations is `7*10^8` , the average difference will be about 6 , it's quiet tiny to store .

Use `Binary Indexded Tree` , we turn `a[x]` into `a[x]-a[x-lowbit(x)]` , so the average of array `a` turns into `6*log(n)/2` .

We can just save the low 8 bits of each element , for the elements which has higher bits than 8 , we can create a new table to storage it . Actually , there is 10359916 elements in the new table , takes `69MiB` disk space .

Finally we can query the phone number through QQid , all the file we must save sizes `3.4GiB` in total .

# Bidirection mapping

Oh , I want to query the QQid through phone number also !

The most direct idea maybe swap two values , and do it again . But it takes more disk space .

There's a good idea says , just record `b[x]` as the address of the phone number which ranks `x` . When dichotomous phone number , just goto `b[mid]` to check if its too big or small .
This part needs a new file saves the ranking . It will be about `2.7GiB` . 

Total space `6.1GiB` supporting O(log n) query the value bidirection .
