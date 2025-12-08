# Similarities
______
______
## Object Themes
The most dominant objects by far are animals, particularly dogs and cats. This pretty much suggests model bias, as the model is likely trained on many instances of cats and dogs. More importantly, some representation of combination of features in FPN layers is very dominant (many of these high correlation images have 15 to 20% explained variance through the top component).

But FasterRCNN + FPN are trained on the same images as SALICON dataset (COCO images). It could just be that humans and models find faces salient, no matter what. Specifically, of all the animal images, it is quite apparent that faces dominant saliency.

Examples below:
![2nd Highest](02.png)

![7th Highest](07.png)

![17th Highest](17.png)
______
______
## Scene Cluttering
One of the key advantages of FPNs is better ability to process distant scenes. In particular, the model can combine "big picture" scenes with large number of objects detected in the background because the receptive field is a combination of various spatial sizes, from fine to coarse. Since the default downsamples all FPN layers to 8x10 spatial regions, the receptive field is huge (albeit blurry). Nevertheless, the model can still delineate the contrast between background and salient foreground.

Objects that are similar but larger have greater saliency. This is apparent in the image of the tanker and cars at a distant overhead angle. To declutter the scene, this combination of FPN layers detect similar objects (cars, tankers), but places heavy emphasis on the larger object due to larger contrast. This tracks with the architecture of the model, where the 8x10 pool dimensions dominate.

Examples below:
![20th Highest](20.png)

![23rd Highest](23.png)
