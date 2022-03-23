---
template: BlogPost
path: /bumpit
mockup: 
thumbnail: 'https://res.cloudinary.com/practicaldev/image/fetch/s--2AKSowW9--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://crunchingnumbersdotlive.files.wordpress.com/2018/08/monte_carlo_simulation.png'
github: https://github.com/Raanaji/MATLAB-Projects-Autumn-Spring-2018-2019
date: 2019-04
name: Finite Difference Method using Monte Carlo Simulations
title: This app got us first place at a Microsoft Azure Hackathon.
category: Microsoft Hackathon
description: '1st place app idea would leverage latest of Azure and ML to support a $6.4 billion infrastructure issue.'
tags:
  - Azure
  - Jupyter
---
## The Challenge

The main objective of the competition revolved around developing innovative solutions for improving the infrastructure of cities. The [Smart Infrastructure Hackathon](https://www.eventbrite.com/e/smart-infrastructure-hackathon-tickets-76918610635#) was hosted at The Cannon, in partnership with Microsoft. Thankfully, we had a team of really bright cloud architects for guidance.

After juggling several ideas we settled with the issue of road potholes. We found that potholes cost American drivers $6.4 billion dollars per year, that is in both repair and insurance cost's. Additionally, the average price to repair a pothole including labor is about $30 - $50 per pothole, while the damage that single pothole may cause are up to $300/year per vehicle ([2016](https://www.pothole.info/2016/05/so-many-potholes-so-much-cost/)). Thus the effort to aiding this issue can be indeed cost-effective.



```latex
\begin{gathered}
x^2 = 3 + \int
\end{gathered}
```



## Our Proposal

The idea of the application to be a simple as possible to minimize driving distractions. Therefore, we intended to create an interface that would show a dialog box only when our system predicted there to be a pothole, from there the user would confirm if so. Also, we designed for a 2-layer confirmation of a road pothole, one by the active camera, and secondly using either the user's phone telemetry data.

## Analyzing Mobile Data

Every modern smartphone is equipped with very accurate telemetry systems. For instance the GPS and accelerometer, which we leveraged for location and to track any abrupt movement indicating a pothole on the road. One of the difficulties with this approach was that logging every high acceleration movement may result in false positives of a road pothole.

| Datetime |  Accel.X (m/s²) | Accel.Y (m/s²) | Accel.Z (m/s²) | Latitude | Longitude | Altitude (m) |
| --- | --- | --- | --- | --- | --- | --- |
2019-12-14 16:28:30:975 | -1.03 | -0.18 | 9.69 | 29.7955 | -95.568 | 6
2019-12-14 16:28:31:475 | -1.01 | -0.03 | 9.70 | 29.7955 | -95.568 | 6
2019-12-14 16:28:31:975 | -1.07 | -0.02 | 9.67 | 29.7955 | -95.568 | 6
2019-12-14 16:28:32:475 | -0.91 | -0.03 | 9.75 | 29.7955 | -95.568 | 6
2019-12-14 16:28:32:975 | -0.96 | -0.20 | 9.67 | 29.7955 | -95.568 | 6

![accelerometer data](assets/bumpit/fig1.svg)
<figcaption>Smartphone accelerometer data</figcaption>
