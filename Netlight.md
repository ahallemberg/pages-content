*Aug 2026 · 4 min read*

I spent the summer of 2026 as an intern at Netlight, a Nordic technology consultancy. The internship lasted six weeks. I was on the data and AI side of the intern group, and was placed on a project for the World Food Programme's Innovation Accelerator together with one other intern.

## The project: temporary settlement detection for SKAI+
[SKAI](https://innovation.wfp.org/project/SKAI) is WFP's platform for reading satellite imagery with machine learning instead of sending a team out to look. It started as a tool for assessing building damage after disasters, and it has grown since then. WFP's own project page mentions that the temporary settlement detection model under SKAI+ reached MVP stage and was planned for pilot deployment with country offices.

That model is what we worked on. The task is to find tents and temporary shelters in high resolution satellite imagery, so analysts can estimate the size of a displaced population without travelling there.

When we started, the detector was a YOLO11 medium model that drew boxes around individual shelters. It worked well in the area it had been trained on. Outside that area it fell apart.

## What was actually wrong
Poor performance on new areas was expected. What was harder to explain was that the model also did badly in parts of the same region that had not been labelled, areas that looked more or less identical to the training data.

Our first instinct was that the model was too small, or that something had shifted in the imagery. We went and looked at the annotations instead, and that turned out to be the whole story. The dataset was small, but the bigger issue was that it was only partly labelled. Inside tiles that had labels, a large share of the real shelters had no box on them at all. The boxes that did exist were often loose around the object.

Missing labels are worse than missing data. A shelter with no box is a real object that the model is being told is background. We were not just failing to teach it what a shelter looks like. We were training it to ignore them.

## What we did
The biggest change was relabelling. Doing the whole area by hand was not realistic in six weeks, so we built a semi automatic pipeline and spent a lot of time iterating on it. It used geometry derived from the statistics of the boxes a human had already drawn, together with a few models trained to propose candidate regions and score how much a candidate looked like a verified shelter. We ended up with full coverage of the area and roughly ten times as many annotations as we started with. Nothing else we did came close to this in effect.

The targets are around ten pixels across. A normal YOLO backbone downsamples enough that objects that small are mostly gone before they reach the detection heads, so we added a P2 head and let the model predict from a shallower, higher resolution level of the feature pyramid.

That created a new problem. Ultralytics ships the P2 variant as a config file only, with no pretrained weights, and only about half of the COCO weights from the standard medium model fit the modified network. So we pretrained on three public aerial datasets combined, oversampling images that contained objects in the same pixel range as our targets. That gave the new head a reasonable starting point. Pretraining from scratch is not something you do on a laptop, so all our training runs went on an HPC cluster.

We also moved from YOLO11 to YOLO26, which was a bit better out of the box and brought small object aware label assignment into the training recipe. For a problem made entirely of small objects that seemed worth having.

We increased the tiling window from 256 to 640 pixels. Settlements cluster heavily, and a wider window lets the model use that structure instead of seeing one isolated speck. It also means fewer shelters get cut in half at a tile edge.

Since transfer to new areas was the original complaint, we brought in a second dataset from a different place for finetuning, and resampled it so the ground sample distance matched our main imagery. We wanted the model to learn what a shelter looks like rather than what one particular place looks like.

Finally we added segmentation. A box is a poor description of a shelter footprint, especially when the shelter is not rectangular or sits at an angle to the image axes. Autolabelling masks for objects this small is not realistic, so I built a small web app that let the other interns contribute masks, and we trained a segmentation model on what came out of it.

## Results
On held out regions that were never part of training, the new model found more than twice as many shelters at the default confidence threshold. Recall went up a lot and precision went up as well, which is the clearest sign that the labels really were the problem. The segmentation model gave footprints that were much tighter than boxes, especially on irregular shapes.

We did not get everything into production inside six weeks. The work was handed over with documentation, and the results looked good enough that I expect it to continue.

## What I learned
The useful lesson was about diagnosis rather than technique. My instinct at the start was to reach for the model. Newer architecture, more capacity, better augmentation. The real bottleneck was sitting in the annotation files, and no amount of architecture work would have fixed it. Spending the first week looking at data instead of at training curves is probably the most transferable thing I took away.

The other lesson was about constraints that are not technical. A lot of what we built was shaped by what could be verified, documented and handed to someone else in a short window. Building something useful for an organisation you are going to leave in six weeks is a different job from building something that works.
