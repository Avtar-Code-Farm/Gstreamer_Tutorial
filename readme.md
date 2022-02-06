# Gstreamer
#Learning
#Learning/Gstreamer

## What is Gstreamer?
Gstreamer is an open source multimedia framework mainly used to create media application (streaming, media plyaback, non-linear editing etc.).  It uses plugins or elements that provide the various codec and other functionalities. 

## What is Codec?
Co -> Commpress
Dec -> decompress

Codec helps in capturing, storing, transferring video content. There are many type of codec and we use them based on the level of media processing stage. 

## What is element?
The Elements are Gstreamer basic construction blocks. They process the data as it flows /downstream/ from the source element (data producer) to the sink element (data consumers). 

 
![](Gstreamer/1347509A-808A-4A65-A0EE-70C7F807ADC6.png)

### Construct element

```
source = gst_element_factory_make("videotestsrc", "source");
sink = gst_element_factory_make("autovideosink", "sink");

```


![](Gstreamer/B876F75C-2305-4A63-BE86-673EF7A95695.png)

### Create pipeline
```
 /* Create the empty pipeline */
  pipeline = gst_pipeline_new ("test-pipeline");

```

All element in the Gstreamer must be part of the pipeline before they can be used. 

### Build pipeline

```
/* First add element into the pipeline */
gst_bin_add_many(GST_BIN(pipeline), source, sink, NULL);  // mind the cast here : GST_BIN (pipeline)
/* Then link them */
gst_element_link(source, sink); // Keep in mind that only elements residing in the same bin can be linked together, so remember to add them to the pipeline before trying to link them!
```

### What is bin?

A pipeline is type of bin therefore all methods which apply to bins also apply to pipelines. Now question is what is bin? -> Bins allow you to combine a group of linked elements into one logical element. You do not deal with the individual elements anymore but with just one element, the bin. Here we treat the pipeline in the similar fashion. 


### What is GObject and properties?
All element/bin are derived from GObject which is the base object. GObject offer set of properties which we can set against element and bin. 

Most GStreamer elements have customizable properties: named attributes that can be modified to change the element’s behavior (writable properties) or inquired to find out about the element’s internal state (readable properties).

Properties are read from with  [g_object_get](https://developer.gnome.org/gobject/unstable/gobject-The-Base-Object-Type.html#g-object-get) () and written to with  [g_object_set](https://developer.gnome.org/gobject/unstable/gobject-The-Base-Object-Type.html#g-object-set) ().

Note here: these property set and get method start with `g_` because these are used to set the properties of GObject.


### Create element using factory
` source = gst_element_factory_make(“videotestsrc”, “source”);`
`  sink = gst_element_factory_make(“autovideosink”, “sink”);`

### Create pipeline,  add elements and then  Link elements 
```
pipeline = gst_pipeline_new(“test-pipeline”);
gst_bin_add_many(GST_BIN(pipeline), source, sink, NULL);
gst_element_link(source, sink)

```

### Set pipeline to play state
```
gst_element_set_state(pipeline, GST_STATE_PLAYING)
```



Code which connects three elements and produce video:

```
#include <gst/gst.h>

int main(int argc, char *argv[])
{
  GstElement *pipeline, *source, *sink, *vertigotv;
  GstBus *bus;
  GstMessage *msg;
  GstStateChangeReturn ret;
  GMainLoop *main_loop;

  /* Initialize GStreamer */
  gst_init(&argc, &argv);

  /* Create the elements */
  source = gst_element_factory_make(“videotestsrc”, “source”);
  sink = gst_element_factory_make(“autovideosink”, “sink”);
  vertigotv = gst_element_factory_make(“vertigotv”, “vertigotv”);

  /* Create the empty pipeline */
  pipeline = gst_pipeline_new(“tesst-pipeline”);

  if (!pipeline || !source || !sink)
  {
    g_printerr(“Not all elements could be created.\n”);
    return -1;
  }

  /* Build the pipeline */
  gst_bin_add_many(GST_BIN(pipeline), source, vertigotv, sink, NULL);

  if (gst_element_link(source, vertigotv) != TRUE)
  {
    g_printerr(“Elements could not be linked.\n”);
    gst_object_unref(pipeline);
    return -1;
  }
  if (gst_element_link(vertigotv, sink) != TRUE)
  {
    g_printerr(“Elements could not be linked.\n”);
    gst_object_unref(pipeline);
    return -1;
  }

  /* Modify the source’s properties */
  g_object_set(source, “pattern”, 0, NULL);

  /* Start playing */
  ret = gst_element_set_state(pipeline, GST_STATE_PLAYING);
  if (ret == GST_STATE_CHANGE_FAILURE)
  {
    g_printerr(“Unable to set the pipeline to the playing state.\n”);
    gst_object_unref(pipeline);
    return -1;
  }
  main_loop = g_main_loop_new(NULL, FALSE);
  g_main_loop_run(main_loop);

  /* Wait until error or EOS */
  bus = gst_element_get_bus(pipeline);
  msg =
      gst_bus_timed_pop_filtered(bus, GST_CLOCK_TIME_NONE, (GstMessageType)(GST_MESSAGE_ERROR | GST_MESSAGE_EOS));

  /* Parse message */
  if (msg != NULL)
  {
    GError *err;
    gchar *debug_info;

    switch (GST_MESSAGE_TYPE(msg))
    {
    case GST_MESSAGE_ERROR:
      gst_message_parse_error(msg, &err, &debug_info);
      g_printerr(“Error received from element %s: %s\n”,
                 GST_OBJECT_NAME(msg->src), err->message);
      g_printerr(“Debugging information: %s\n”,
                 debug_info ? debug_info : “none”);
      g_clear_error(&err);
      g_free(debug_info);
      break;
    case GST_MESSAGE_EOS:
      g_print(“End-Of-Stream reached.\n”);
      break;
    default:
      /* We should not reach here because we only asked for ERRORs and EOS */
      g_printerr(“Unexpected message received.\n”);
      break;
    }
    gst_message_unref(msg);
  }

  /* Free resources */
  gst_object_unref(bus);
  gst_element_set_state(pipeline, GST_STATE_NULL);
  gst_object_unref(pipeline);
  return 0;
}
```




![](Gstreamer/AC1DD64D-3863-4C79-A964-F42D08A7096F.png)



### Important learning here:

    create elements with gst_element_factory_make()

    create an empty pipeline with gst_pipeline_new()

    add elements to the pipeline with gst_bin_add_many()

    link the elements with each other with gst_element_link()




- - - -
Chapter 3 - **Dynamic pipelines**
- - - -

First learn few concepts:

**Multiplexed**: A container file which contains audio and video. 
**Container** format: MKV, QT, MOV, Ogg or advance format wma wmv ASF
**Demuxers**: Element responsible for opening such container file is called demuxers.

A container embeded multiple streams. The demuxer  will separate them and expose them through diff port. Different branches can be created in the pipeline, dealing with diff type of data.

**GstPad** : The ports through with Gstreamer element communicate with each other are called GstPad. There exists `Sink Pads` , through which data flow into the element and there exists `Source Pads` , through which data flow out of the element. 

 *source elements* contain only src pad 
*Sink elements* contain only sink pad
*filter elements* contain both pad (sink and src)
*Demuxer Elements” contain three pads -> sink & audio & video  
		- **audio pad** can provide input to another element sink’s pad to process audio
		- **video pad** can provide input to another element sink’s pad to process video. 


![](Gstreamer/6D3AE027-3C90-4173-A633-EFF01BA41E55.png)



Use case of dynamic pipeline: Demuxer is not aware of how many src pad it needed until it inspect the incoming data. After identifying the type of the media in the incoming data, we can append the element for each src pad coming out of demuxer. 

## Signals
```
/* Connect to the pad-added signal */
g_signal_connect (data.source, "pad-added", G_CALLBACK (pad_added_handler), &data);
```

GSignals are a crucial point in GStreamer. They allow you to be notified (by means of a callback) when something interesting has happened. Signals are identified by a name, and each GObject has its own signals.


### Gstreamer State

Null  -> Ready -> Paused -> Playing 

NULL -> The NULL state or initial state
READEY -> The element is ready to go to PAUSED
PAUSED -> The element is ready to accept and process data. 
PLAYING -> The element is playing and click is running and data is flowing



You can move to the adjacent state however cannot move across state i.e moving from NULL to PLAYING state is not valid. 



##  Media formats and Pad Capabilities

Pad capabilities are a fundamental part of Gstreamer. 

### Pads allow information to enter and leave an element. The capabilities (or Cap for short) of the pad specify what kind of information can travel through the pad. 

Example of Cap

1. RGB video with resolution of 320x200 pixels and 30 FPS
2. 16-bits per ample audio, 5.1 channel at 44100 samples per second 
3. Mp3 or h264 (compress format)

In order for two elements to link together, they must share a common subset of capabilities; otherwise they couldn’t possibly understand each other. This is the main goal of the capabilities. 



### Pad Templates

Pads are created from Pad templates, which indicate all possible capabilities a pad could ever have. Template are useful for creating several similar pads, and allow early refusal of connection b/w elements: if the Capabilities of their PAD templates do not have a common subset. 

Capabilities examples

```
SINK template: 'sink'
  Availability: Always
  Capabilities:
    audio/x-raw
               format: S16LE
                 rate: [ 1, 2147483647 ]
             channels: [ 1, 2 ]
    audio/x-raw
               format: U8
                 rate: [ 1, 2147483647 ]
             channels: [ 1, 2 ]

```


























