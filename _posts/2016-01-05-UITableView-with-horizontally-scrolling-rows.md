---
layout: post
title: UITableView with horizontally scrolling rows
---

## Introduction

The biggest layouting decision of any ```UITableView``` is properly categorising the content so it makes sense to the user. This is often achieved by using sections with headers and footers. While this approach is extremely easy to implement and clear, often times it's either not suitable for the type of content or it looks boring-ish

One of the better solutions I've found is the horizontally scrolling cells within the ```UITableView```. The best example that currently comes to my mind is the Netflix app

![Netflix app](http://a3.mzstatic.com/us/r30/Purple7/v4/b1/78/29/b17829ff-9c83-e0b9-61ae-51ad1f69ec1a/screen322x572.jpeg)

Using the horizontally scrolling cells of the ```UITableView``` enables us to develop a highly intuitive interface. It also enables the user to glance a lot of information (9 titles in the picture above).

## Storyboarding
Firstly you're going to need to add a ```UITableViewController``` and within the ```UITableViewController``` and it's `UITableView`  subview, you need to add the `UITableViewCell` if it wasn't already added by the Interface Builder.  
Strech the `UITableViewCell` to be the height you want it to be (250 in this exmaple)

![im](http://i.imgur.com/WzxB9Fd.png)

After that, you're going to need to add an instance of `UICollectionView` inside the `UITableViewCell`. 

Then, strech the `UICollectionView` to match the size of the `UITableViewCell`. Once you've done that, use autolayout to set the constraints so that the `UICollectionView` has the same position and size as the `UITableViewCell`.

![img](http://i.imgur.com/dasGDpV.png)

After that, you're going to need to go and create a new class for the `UITableViewController`.  Go to File->New->File->CocaTouch Class and then select `UITableViewController` in the dropdown menu. 
After you have completed that step, you need to connect the new class to the storyboard instance of the `UITableViewController`. Go to your .storyboard and select the `UITableViewController` and in the upper right corner select the icon that, on hover, says "Identity Inspector". 
In the Identity Inspector, under "Custom Class", set the "Class" attribute to the name you just gave your class.


Now create two new classes for the cells. For the first one, go to File->New->File->CocoaTouch Class and then select `UITableViewCell` in the dropdown. Go to the storyboard, select the `UITableViewCell` and connect it the same way you connected the `UITableViewController`. 
For the second one, go to File->New->File->CocoaTouch Class and select `UICollectionViewCell`. Now go to the storyboard, select the CollectionView Cell and connect it like you did with the previous two widgets. 

## Code

In the file you created for the `UITableViewController` override the following methods

{% highlight swift %}
	
override func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCellWithIdentifier("CollectionHolderTableViewCell") as! CollectionHolderTableViewCell

/*
	NOTE: Add a "model" field inside the CollectionHolder cell and send any needed info into it
*/
        return cell
}

override func numberOfSectionsInTableView(tableView: UITableView) -> Int {
        //Here you would use the amount of groups in the "model"
        return 1
}

override func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        //Here you would use the amount of items in every group in the     //"model"

        return 5
}

{% endhighlight %}

Now that you've finished implementing teese methods, feel free to run the app!
Nothing will happen because, although we have implemented the creation of the `UITableView` and it's cells, we still havent created the `UICollectionView` and it's cells. Since we dragged the `UICollectionView` outlet into the instance of our `UITableViewCell `class, we will access it from there. 

Firstly, make sure that the `UITableViewCell` class declaration looks like this

``` swift
class CollectionHolderTableViewCell: UITableViewCell, UICollectionViewDataSource 
```

Adding the `UICollectionViewDataSource` will enable us to give data to the `UICollectionViewCell`'s . In order to make sure that the `UICollectionView` is listening to the methods we will implement in this class, we need to set it's `dataSource` property to `self`. Override the following method

``` swift
override func layoutSubviews() {
    dogeCollectionView.dataSource = self
    //NOTE dogeCollectionView is the name I gave to the collectionView when making the outlet
}
```

Now the `UICollectionView` will listen for the implementation of a specific method in our class. Implement that method as follows:

``` swift
func collectionView(collectionView: UICollectionView, cellForItemAtIndexPath indexPath: NSIndexPath) -> UICollectionViewCell {
    let cell = collectionView.dequeueReusableCellWithReuseIdentifier("dogeCell", forIndexPath: indexPath) as! DogeCollectionViewCell
    /*
        It's called doge.jpeg since I added an image of Shiba Inu in the project assets folder. You would import some information from a REST API or similar
    */
    cell.dogeImageView.image = UIImage(named: "doge.jpeg")
    
    return cell
}

func collectionView(collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
    //You would get something like "model.count" here. It would depend on your data source
    return 5
}

func numberOfSectionsInCollectionView(collectionView: UICollectionView) -> Int {
    return 1
}
```

![doge](http://i.imgur.com/IyXdzye.png)

If you did everything properly, you should have a `UITableViewController` with horizontally scrolling cells

## Clicking and transition

All transitions need to be done using segues. Calling `navigationController.pushViewController()` won't work from within 2 layer of depth. You are also advised to disable the selection on the `UITableView`, but that depends on what implementation you need.