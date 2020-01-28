---
layout: post
title:  "Using K-Means Clustering to generate color palette for images"
abstract: "This post tells you what is k-means clustering, when to use it and how to use it, with example code snippets. It also includes an explanation on how to fine-tune the results you’ll get."
date:   2020-01-25 14:38:32 +0100
categories: machine-learning
blabla: "dd"
---

<h4>Intro</h4>

Recently I’ve found an instagram profile dedicated to color palettes used in various movie scenes. See here:
<a href="https://www.instagram.com/colorpalette.cinema/">colorpalette.cinema</a>.

I started wondering if there’s a way to generate those palettes automatically. I know that one could pick them manually since it’s only a few colors per image. But there are also quite nice and relatively simple methods of letting the computer do it. I tried one of them and these are the results:

<div>
<img src="{{site.baseurl}}/assets/img/kmc-orm.jpg" style="width:50%"/>
<img src="{{site.baseurl}}/assets/img/kmc-pdwl.jpg" style="width:50%; float: left"/>
{% for color in site.data.k-means.kmc-pdwl %}
  <div style="float:left; width: 6.25%; height: 35px; outline: 1px solid black; outline-offset: -1px; background-color: {{color}}"></div>
{% endfor %}
{% for color in site.data.k-means.kmc-orm %}
  <div style="float:left; width: 10%; height: 35px; outline: 1px solid black; outline-offset: -1px; background-color: {{color}}"></div>
{% endfor %}
</div>
<br><br>
<div>
<img src="{{site.baseurl}}/assets/img/kmc-sgh.jpg" style="width:50%"/>
<img src="{{site.baseurl}}/assets/img/kmc-iz.jpg" style="width:50%; float: left"/>
{% for color in site.data.k-means.kmc-iz %}
  <div style="float:left; width: 7.14%; height: 35px; outline: 1px solid black; outline-offset: -1px; background-color: {{color}}"></div>
{% endfor %}
{% for color in site.data.k-means.kmc-sgh %}
  <div style="float:left; width: 8.33%; height: 35px; outline: 1px solid black; outline-offset: -1px; background-color: {{color}}"></div>
{% endfor %}
</div>

To understand how this was achieved, read below.

<h4>What is K-Means Clustering?</h4>

It’s an algorithm used to solve clustering problems (the <a href="https://en.wikipedia.org/wiki/NP-hardness">NP-hard</a> ones). It’s relatively simple and also an example of unsupervised learning algorithm.

<h4>When do we use it?</h4>

When we have a set of some data points that have features and values. We want to group data points into clusters based on these features. Real-life example: extracting leading colors from images.


<h4>How does it work?</h4>

We start by randomly picking k data points (k is set beforehand) called centroids from our data set. All remaining points are then assigned to one of these centroids. Assignment is done based on the distance - closest centroid is picked. For each centroid and its data points we calculate average value - a new centroid. The process of assign - calculate distance - update centroids is repeated n times, until a desired result is reached. How to know if we achieved that - this will be covered later on in this article.

Let’s look into the details:

<h4>Step 1 Read Data</h4>

In our case, the input data set is a .jpg file. We’ll have to extract each pixel’s `value` from the image. We’ll use a `getRGB(int x, int y)` method from Java’s `BufferedImage`.

{% highlight java %}
public class Point {
   private Integer value;
   …
}
{% endhighlight %}

This is our data point class. It’s only one field value stores the rgb value of each pixel in the image.

<h4>Step 2 Initialize centroids</h4>

The next step is picking random K points from data points list. These random points will be our centroids.

{% highlight java %}
public List<Centroid> initializeCentroids(int k, BufferedImage image){

   if(image == null){
       throw new IllegalArgumentException("Buffered image cannot be null");
   }

   List<Centroid> initialCentroids = new ArrayList<>();

   int xAxes = image.getWidth();
   int yAxes = image.getHeight();

   for(int i = 0; i < k; i++) {
       int coordX = new Random().nextInt(xAxes);
       int coordY = new Random().nextInt(yAxes);
       Integer rgb = image.getRGB(coordX, coordY);
       initialCentroids.add(new Centroid(rgb));
   }
   return initialCentroids;
}
{% endhighlight %}

<h4>Step 3 Calculate distance</h4>

Now we need to iterate over our data points, find the nearest centroid and assign the point to that centroid.

{% highlight java %}
public Map<Centroid,List<Point>> reassignPointsToNearestCentroids(List<Point> points, Map<Centroid,List<Point>> centroidsToListOfPoints){

   List<Centroid> currentCentroids = new ArrayList<Centroid>(centroidsToListOfPoints.keySet());

   for (Point point : points) {
       Centroid nearest = getNearestCentroid(point, currentCentroids);
       if (centroidsToListOfPoints.get(nearest) == null) {
           centroidsToListOfPoints.put(nearest, new ArrayList<>(Arrays.asList(point)));
       } else {
           centroidsToListOfPoints.get(nearest).add(point);
       }
   }

   return centroidsToListOfPoints;
}

public Centroid getNearestCentroid(Point point, List<Centroid> centroids) {

   if(point == null){
       throw new IllegalArgumentException("Point cannot be null");
   }
   if(centroids == null || centroids.isEmpty()){
       throw new IllegalArgumentException("List of centroids cannot be null or empty");
   }

   double minimumDistance = Double.MAX_VALUE;
   Centroid nearest = null;

   for (Centroid centroid : centroids) {
       Distance distance = new Distance2D();
       double currentDistance = distance.getDistance(point.getValue(), centroid.getValue());

       if (currentDistance < minimumDistance) {
           minimumDistance = currentDistance;
           nearest = centroid;
       }
   }
   return nearest;
}

public class Distance2D extends Distance {

   @Override
   public Double getDistance(Double a, Double b) {
       return Math.abs(a - b);
   }
}
{% endhighlight %}

<h4>Step 4 Calculate means</h4>

For each centroid and its points, calculate mean value.

{% highlight java %}
public Map<Centroid, Centroid> computeKNewCentroids(Map<Centroid,List<Point>> centroidsToListOfPoints){

   Map<Centroid, Centroid> oldAndNewMap = new HashMap<>();
   for (Centroid oldCentroid : centroidsToListOfPoints.keySet()) {
       List<Point> points = centroidsToListOfPoints.get(oldCentroid);
       Integer average = Mean.calculateForPoints(points);
       Point nearest = getNearestPoint(new Point(average), points);
       Centroid newCentroid = new Centroid(nearest.getValue());
       oldAndNewMap.put(oldCentroid, newCentroid);
   }
   return oldAndNewMap;
}
{% endhighlight %}

<h4>Step 5 Update centroids</h4>

The mean value that we’ve calculated in the previous point is our new centroid. We’ll have to update all of the centroids to the new value.

{% highlight java %}
public Map<Centroid,List<Point>> updateCentroids(Map<Centroid,List<Point>> centroidsToListOfPoints, Map<Centroid, Centroid> oldAndNewMap){
   for (Centroid oldCentroid : oldAndNewMap.keySet()){
       Centroid newCentroid = oldAndNewMap.get(oldCentroid);
       if(newCentroid.getValue() != oldCentroid.getValue()){
           List<Point> points = centroidsToListOfPoints.remove(oldCentroid);
           centroidsToListOfPoints.put(newCentroid, points);
       }
   }

   return centroidsToListOfPoints;
}
{% endhighlight %}

<h4>Repeat steps 3-5</h4>

Repeat those steps until desired result is reached.

<h4>Determine the optimal value of K</h4>

So we want to process an image like this:

<img src="{{site.baseurl}}/assets/img/kmc-expo.jpg" style="width: 100%">

Fine, but how can we determine which value of k is the best one here?

First, let's try setting k values from 2 to 8 on this image. These are the results:

{% for color in site.data.k-means.k2 %}
  <div style="float:left; width: 6.25%; height: 35px; outline: 1px solid black; outline-offset: -1px; background-color: {{color}}"></div>
{% endfor %} k = 2
<br><br>
{% for color in site.data.k-means.k3 %}
  <div style="float:left; width: 6.25%; height: 35px; outline: 1px solid black; outline-offset: -1px; background-color: {{color}}"></div>
{% endfor %} k = 3
<br><br>
{% for color in site.data.k-means.k4 %}
  <div style="float:left; width: 6.25%; height: 35px; outline: 1px solid black; outline-offset: -1px; background-color: {{color}}"></div>
{% endfor %} k = 4
<br><br>
{% for color in site.data.k-means.k5 %}
  <div style="float:left; width: 6.25%; height: 35px; outline: 1px solid black; outline-offset: -1px; background-color: {{color}}"></div>
{% endfor %} k = 5
<br><br>
{% for color in site.data.k-means.k6 %}
  <div style="float:left; width: 6.25%; height: 35px; outline: 1px solid black; outline-offset: -1px; background-color: {{color}}"></div>
{% endfor %} k = 6
<br><br>
{% for color in site.data.k-means.k7 %}
  <div style="float:left; width: 6.25%; height: 35px; outline: 1px solid black; outline-offset: -1px; background-color: {{color}}"></div>
{% endfor %} k = 7
<br><br>
{% for color in site.data.k-means.k8 %}
  <div style="float:left; width: 6.25%; height: 35px; outline: 1px solid black; outline-offset: -1px; background-color: {{color}}"></div>
{% endfor %} k = 8
<br><br>

We can see that for small k values colors seem quite random and every time value of k gets increased, the color set feels more like matching the image's palette. It's also visible that the variation between colors gets smaller and smaller. By looking at these samples we can actually see it and recognize it, but how would we do that if out features were not colors, but some other data types?

To determine optimal value of k, we'll use an `elbow method`. We'll plot clusters' average dispersion (standard deviation in our case) and increasing k values. 

<img src="{{site.baseurl}}/assets/img/elbow-graph.svg" style="width: 100%" alt="Elbow method for optimal k">

We can see that standard deviation decreases with rising k value. We could also see that by looking at colors. What we need is the value of k, where the deviation decreases the most - this is called `elbow point`. On the graph we can see that it's happening when value of k is 5.

<h4>Conclusion</h4>

With k-means clustering we can group our input data set into clusters. This method is quite easy to understand and to use. It's an example of centroid-based clustering algorithms. Other types of algorithms are connectivy-based, distribution-based and density-based ones. 

There are many ways of calculating statistical dispersion for clusters. I've chosen standard deviation, but variation or interquartile range can also be used. 

See full code at <a href="https://github.com/madhr/kmeans-clustering">github</a>