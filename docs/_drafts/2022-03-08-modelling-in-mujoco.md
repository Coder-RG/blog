---
layout: post
title: Modelling in Mujoco's XML format
categories: reinforcement-learning
---

Oh hello!👋

Welcome to this article on modelling in [MuJoCo](https://mujoco.readthedocs.io/).
Hopefully, this article will provide you with a good understanding of modelling
in XML format, so that you may create many more complex models of your own.

I will show you by creating a Quadrotor in Mujoco which requires the use of actuators
and sensors beyond bodies and geoms.

Let's start by defining mujoco model. Use the `model` parameter for naming your model.
The rest of the elements will be placed inside this structure.

```xml
<mujoco model='quadrotor'>
    <!-- Body -->
</mujoco>
```

You will only see black background. We can change that by using `asset` command.

```xml
<mujoco model='quadrotor'>
    <asset>
        <texture type="skybox" builtin="gradient" rgb1="1 1 1"
            rgb2=".6 .8 1" width="256" height="256"/>
    </asset>
</mujoco>
```


{% capture image_path %}
{{site.baseurl}}/assets/images/mujoco/blue-bg.png
{% endcapture %}

{% capture mycaption %}
Fig {% increment fig_counter %}: Blue Gradient background.
{% endcapture %}

{% include image.html url=image_path caption=mycaption %}

Now, let's create a floor. First we will create a worldbody, which acts
as the parent to all others bodies.

```xml
<mujoco model='quadrotor'>
    <asset>
        <texture type="skybox" builtin="gradient" rgb1="1 1 1"
            rgb2=".6 .8 1" width="256" height="256"/>
    </asset>
    <worldbody>
        <geom name='floor' type='plane' size='5 5 .1' rgba='1 0 0 1'/>
    </worldbody>
</mujoco>
```

{% capture image_path %}
{{site.baseurl}}/assets/images/mujoco/floor.png
{% endcapture %}

<!-- {% capture mycaption %}
Fig {{fig_counter}}: Adding floor to our model
{% endcapture %} -->

{% capture mycaption %}
Fig {% increment fig_counter %}: Adding floor our model
{% endcapture %}

{% include image.html url=image_path caption=mycaption %}

Geoms are used for collision calculation. The shape is chosen using the `type` parameter.
`size` gives the dimensions of the geom. You only specify the half values in size.
The values are of the form `x y z`. It spans both positive and negative axis equally. 
The values `5 2 .1` means 5 in +x and 5 in -x direction. Similar for other axes as well.

{% capture image_path %}
{{site.baseurl}}/assets/images/mujoco/dimensions.png
{% endcapture %}

{% capture mycaption %}
Fig {% increment fig_counter %}: Assigning dimensions in mujoco
{% endcapture %}

{% include image.html url=image_path caption=mycaption %}

| Name | Description | Value |
|:----:|:-----------:|:-----:|
| body | Objects that have inertia and mass but no geometry | None |
| geom | Used to give shape to bodies | predefined object shapes to choose from |
| rgba | Specify the colour of objects | integer; 0-1 |