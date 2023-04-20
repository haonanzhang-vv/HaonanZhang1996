---
title:  "Box and Violin Plot in R with ggplot2"
mathjax: true
layout: post
categories: media
---

![Violin Plot](https://raw.githubusercontent.com/YzwIsALaity/Box-Violin-Plot-Tutorial-in-R/042bda2d9a9dee966d39cdfbc937b63c2de91855/p5.jpeg)


This is a short tutorial for how to change a __plain box plot__ to be more advanced versions (e.g. __box plot with p-value, violin plot__, etc.).
## 1. Format of dataset for boxplot
The dataset is a record of the level of biomarker for a disease and patients in the dataset are categorized into different groups based on their severity levels. Hence, this dataset has three columns:

- `PTID`: __unique identifications for participants__ and there are 400 unique participants (string);

- `Illness`: __severity levels of the disease [4 levels: Healthy Control, Moderate, Mild, Severe]__ (string);

- `Biomarker`: a __numerical measurement of the biomarker__ (numerical);

- `Sex`: a variable for sex of participants (string).

![](https://raw.githubusercontent.com/YzwIsALaity/Box-Violin-Plot-Tutorial-in-R/042bda2d9a9dee966d39cdfbc937b63c2de91855/Dataset_Shape.jpeg)

A __box plot__ is often used to __show the empirical distribution of a numerical covariate regarding different groups__. Usually, there are two main methods to visualize the empirical distribution of the data:

- __Histogram/density__: it approximates the __"shape"__ of an empirical distribution;

- __Box plot__: it provides more __descriptive statistics__ of an empirical distribution in a graphical way. Typically, it has __five elements: min, max, median, 1st & 3rd         quartile__.

Compared to a __histogram/density__, a __box plot__ is more often to use when people want to __compare the same numerical covariate through all groups__ and this process is typically involved in performing __hypothesis testing__. 

## 2. Version 0.0: basic
The first step in here we are going to draw a simple box plot to visualize empirical distribution of `Biomarker` through different groups in `Illness`. In this version, we will __remove the background and grid__ for the box plot. We make __titles in axes in bold type__. The main function for box plot is `geom_boxplot()` and we will set `Illness` in X-axis and `Biomarker` in Y-axis.  

```{r}
# Set Illness and Sex as factors
Dt$Illness <- factor(Dt$Illness, levels = c("Healthy Control", "Mild", "Moderate", "Severe"))
Dt$Sex <- factor(Dt$Sex, levels = c('Male', 'Female'))

# Version 0.0
ggplot(Dt, aes(x = Illness, y = Biomarker)) +
  geom_boxplot(alpha = 0.5) + 
  xlab('Illness Severity') + ylab('Biomarker Level') +
  theme_bw() +                                                      # dark-on-light theme
  theme(panel.border = element_blank(),                             # these four are for the background and grid
        panel.background = element_blank(),                    
        panel.grid.major = element_blank(), 
        panel.grid.minor = element_blank(), 
        axis.line.x = element_line(),                               # these two are for the axis line
        axis.line.y = element_line(),
        axis.text.x = element_text(colour = "black", size = 11),    # there two are for texts in axes
        axis.text.y = element_text(colour = "black", size = 11),
        axis.ticks.x = element_line(),                              # these two are for ticks in axes
        axis.ticks.y = element_line(),
        axis.title.x = element_text(colour = "black", size = 11, face = 'bold', vjust = -1),                              
        axis.title.y = element_text(colour = "black", size = 11, face = 'bold'),
        legend.title = element_text(colour = "black", size = 11, face = 'bold')) 
```

![](https://raw.githubusercontent.com/YzwIsALaity/Box-Violin-Plot-Tutorial-in-R/042bda2d9a9dee966d39cdfbc937b63c2de91855/p1.jpeg)

In Version 0.0, we just show boxes for `Illness` groups and do not stratified them into `Sex`, which provides a general visualization of numerical biomarker levels regarding different `Illness` groups. 

## 3. Version 1.0: grouped
Based on the setting of Version 0.0, the boxplot can be stratified by `Sex` and this only required we pass `fill = Sex` into `aes()` for main ggplot function `ggplot()`. In this version, we will use preselected colors for `Sex`.

```{r}
# we can preselect color for each box 
Col <- c("#FF0000", "#80FF00")
# Version 1.0
ggplot(Dt, aes(x = Illness, y = Biomarker, fill = Sex)) +
  geom_boxplot(alpha = 0.5) +                                       # 'alpha = 0.5' control the transparency of colors
  scale_fill_manual(values = Col) +                                 # use preselected color
  xlab('Illness Severity') + ylab('Biomarker Level') +
  theme_bw() +                                                      # dark-on-light theme
  theme(panel.border = element_blank(),                             # these four are for the background and grid
        panel.background = element_blank(),                    
        panel.grid.major = element_blank(), 
        panel.grid.minor = element_blank(), 
        axis.line.x = element_line(),                               # these two are for the axis line
        axis.line.y = element_line(),
        axis.text.x = element_text(colour = "black", size = 11),    # there two are for texts in axes
        axis.text.y = element_text(colour = "black", size = 11),
        axis.ticks.x = element_line(),                              # these two are for ticks in axes
        axis.ticks.y = element_line(),
        axis.title.x = element_text(colour = "black", size = 11, face = 'bold', vjust = -1),                              
        axis.title.y = element_text(colour = "black", size = 11, face = 'bold'),
        legend.title = element_text(colour = "black", size = 11, face = 'bold')) 
```

![](https://github.com/YzwIsALaity/Box-Violin-Plot-Tutorial-in-R/blob/042bda2d9a9dee966d39cdfbc937b63c2de91855/p2.jpeg)

This is the grouped version of a box plot based on Version 0.0. The above two versions are the most basic type of box plots and __if you want to show the box horizontally, you just need to shift X-axis and Y-axis__. In the next example, we will integrate a box plot with hypothesis testing together.

## 4. Version 2.0: hypothesis testing
To integrate a box plot with hypothesis testing together, we need to use `ggpubr` package. We need to use a new function:

- `stat_compare_means()`: it is used to conduct hypothesis testing among different groups regarding the same numerical covariate. We will introduce some common-used arguments of this function.

  * `method`: it is used to indicate what __types of testing methods__ should be performed (e.g. __`t.test` for normal data, `wilcox.test` for non-normal data, `kruskal.test`, and `anova`__). 
  
  * `label`: it is used to specify label types [e.g. __"p.signif" (shows the significance levels) and "p.format" (shows the formatted p value)__].
  
  * `vjust`: it is used to move the text up or down relative to the bracket.
  
  * `paired`: it is used to indicate whether you want a paired test.

```{r}
# Version 2.0
ggplot(Dt, aes(x = Illness, y = Biomarker, fill = Sex)) +
  geom_boxplot(alpha = 0.5) +                                       # 'alpha = 0.5' control the transparency of colors
  stat_compare_means(aes(group = Sex),                              # 'group = Sex' choose groups for comparison
                     method = 'wilcox.test',                        # we use wilcox test for comparison
                     label = 'p.format', 
                     vjust = -0.65) +
  scale_fill_manual(values = Col) +                                 # use preselected color
  xlab('Illness Severity') + ylab('Biomarker Level') +
  theme_bw() +                                                      # dark-on-light theme
  theme(panel.border = element_blank(),                             # these four are for the background and grid
        panel.background = element_blank(),                    
        panel.grid.major = element_blank(), 
        panel.grid.minor = element_blank(), 
        axis.line.x = element_line(),                               # these two are for the axis line
        axis.line.y = element_line(),
        axis.text.x = element_text(colour = "black", size = 11),    # there two are for texts in axes
        axis.text.y = element_text(colour = "black", size = 11),
        axis.ticks.x = element_line(),                              # these two are for ticks in axes
        axis.ticks.y = element_line(),
        axis.title.x = element_text(colour = "black", size = 11, face = 'bold', vjust = -1),                              
        axis.title.y = element_text(colour = "black", size = 11, face = 'bold'),
        legend.title = element_text(colour = "black", size = 11, face = 'bold')) 
```

![](https://raw.githubusercontent.com/YzwIsALaity/Box-Violin-Plot-Tutorial-in-R/042bda2d9a9dee966d39cdfbc937b63c2de91855/p3.jpeg)

Therefore, it is very easy to integrate a box plot with hypothesis testing! Then, we are going to the final stage of box plot.

## 5. Version 3.0: violin plot
__As we mentioned above, there are two main methods to visualize the empirical distribution of the data--histogram/density and box plot. If we blend them together, that is a violin plot.__ Actually, it is very easy to create a violin plot based on a box plot and the main function we are going to use is 

- `geom_violin()`: this is the main function for creating a violin plot. __If we use this function based on our Version 0.0/1.0/2.0, we just need to adjust the size of "violin"    and "box" with the command `width = `.__

The first example of a violin plot is based on Version 0.0. 

- __The basic logic of creating a violin plot over existed box plot is to put `geom_violin()` at first and then call `geom_boxplot()`.__ 

- __Meanwhile, since the box plot is above the violin plot, we hope violins can cover boxes. It is necessary to adjust the size of violins and boxes to make sure each violin can cover each box entirely.__  

```{r}
# Version 3.0
ggplot(Dt, aes(x = Illness, y = Biomarker)) +
  geom_violin(width = 1, 
              position = position_dodge(0.7)) +                     # each violin has larger size
  geom_boxplot(alpha = 0.7, width = 0.3, 
               position = position_dodge(0.7)) +                    # this size of each box is smaller than a violin
  xlab('Illness Severity') + ylab('Biomarker Level') +
  theme_bw() +                                                      # dark-on-light theme
  theme(panel.border = element_blank(),                             # these four are for the background and grid
        panel.background = element_blank(),                    
        panel.grid.major = element_blank(), 
        panel.grid.minor = element_blank(), 
        axis.line.x = element_line(),                               # these two are for the axis line
        axis.line.y = element_line(),
        axis.text.x = element_text(colour = "black", size = 11),    # there two are for texts in axes
        axis.text.y = element_text(colour = "black", size = 11),
        axis.ticks.x = element_line(),                              # these two are for ticks in axes
        axis.ticks.y = element_line(),
        axis.title.x = element_text(colour = "black", size = 11, face = 'bold', vjust = -1),                              
        axis.title.y = element_text(colour = "black", size = 11, face = 'bold'),
        legend.title = element_text(colour = "black", size = 11, face = 'bold')) 
```

![](https://raw.githubusercontent.com/YzwIsALaity/Box-Violin-Plot-Tutorial-in-R/042bda2d9a9dee966d39cdfbc937b63c2de91855/p4.jpeg)

The second example of a violin plot is based on Version 2.0. __Due to the overlapped of violins and boxes, we need to adjust their overlapped position and we will use one more arguments in both `geom_violin()` and `geom_boxplot()`--`position`. We will set `position = position_dodge()` and pass any some numerical values into `position_dodge()`. `position_dodge()` is used to control the horizontal distance between two groups. Finally, we need to make sure `geom_violin()` and `geom_boxplot()` share the same `position`.__

```{r}
# Version 4.0
ggplot(Dt, aes(x = Illness, y = Biomarker, fill = Sex)) +
  geom_violin(aes(fill = Sex), width = 1, alpha = 0.5,
              position = position_dodge(1)) +     
  geom_boxplot(aes(fill = Sex), alpha = 0.5, width = 0.1, 
               position = position_dodge(1)) +                     
  stat_compare_means(aes(group = Sex),                              # 'group = Sex' choose groups for comparison
                     method = 'wilcox.test',                        # we use wilcox test for comparison
                     label = 'p.format', 
                     vjust = -0.65) +
  scale_fill_manual(values = Col) +                                 # use preselected color
  xlab('Illness Severity') + ylab('Biomarker Level') +
  theme_bw() +                                                      # dark-on-light theme
  theme(panel.border = element_blank(),                             # these four are for the background and grid
        panel.background = element_blank(),                    
        panel.grid.major = element_blank(), 
        panel.grid.minor = element_blank(), 
        axis.line.x = element_line(),                               # these two are for the axis line
        axis.line.y = element_line(),
        axis.text.x = element_text(colour = "black", size = 11),    # there two are for texts in axes
        axis.text.y = element_text(colour = "black", size = 11),
        axis.ticks.x = element_line(),                              # these two are for ticks in axes
        axis.ticks.y = element_line(),
        axis.title.x = element_text(colour = "black", size = 11, face = 'bold', vjust = -1),                              
        axis.title.y = element_text(colour = "black", size = 11, face = 'bold'),
        legend.title = element_text(colour = "black", size = 11, face = 'bold')) 
```

![](https://raw.githubusercontent.com/YzwIsALaity/Box-Violin-Plot-Tutorial-in-R/042bda2d9a9dee966d39cdfbc937b63c2de91855/p5.jpeg)

Eventually, we can get a fancy violin plot with a plain box plot and hypothesis testing! We can also make them horizontally and this just need to exchange a variable in X-axis with a variable in Y-axis as well. 

In here, we just provide two types of combination of plots so it is very possible that people can combine box/violin plots with other plots (e.g. points, bars, etc.). 


