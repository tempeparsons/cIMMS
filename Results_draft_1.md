Results

In this study we applied GMM to data from cIMMS that varied in voltage, time and itensity. The main goal was to establish an automated method by which the number of gaussian peaks (as described by their mean and variance) within the total data for all voltages for a single slice from the original parent protein species could be determined. This would reveal the decomposition stability of the protein in response to voltages disruption. The mean and variation of decomposition peaks would help determine the positions at which the parent protein was most vulnerable to unfolding. 

Show one of the datasets 2D plots in full. Introduce the shape of the data using the hIAPP_A dataset. Shapes of other datasets can be in supplemental. 
First Figure.

Show code block for reading all the data in. 

Clustering methods, including GMM, are most frequently applied to 1D or 2D data. In this case, data is measured in time, voltage and intensity. The dimensionality of the data can be reduced to 2D by representing intensity as the number of points at a particular time point. If this clustering is per per voltage, only 1D clustering is required as shown below.

Second Figure show block of hIAPP_A processed in this way.

When performng GMM in SKL there are a number of parameters to be considered, of which two are properly important. Firstly, there is the method for picking the initial number of peaks, or components, that the EM algorithm will start with. Explain the options. Secondly, there is the covariance type. Explain this.

The optimal number of components can be deduced using BIC, for which there is an inbuilt method in SKL. Explain how it works. This means that a steep drop in BIC will be observed with then number of initial components is optimal. 
For this data I used examined he impacts of both full and diag covariance. Explain why. 

The datasets for separate voltages, each containing a number of datapoints proportional to the internsity at each time point, can be combined for modelling in 2D using the same principles for 1D clustering in Scikitlearn shown above.

The below figure shows the shape of the combined data pre-modelling. Datapoints are set to high transparancy, so darker points represent higher intensity. The high range of intensity measurments is better conveyed using a contour map of the data. 

Show dot hist.

Once the data for the entire hIAPP_sliceA dataset is arranged as a 2D histogram, GMM can be applied. I investigated the impact on BIC of choosing 2 - 6 initial components and setting covarience type to both full and diag. For all these inital components, I examined the success of modelling the peak distributions on by plotting colored peak distributions on to the above 'dot-style' histogram, plotting the propsed means and standard deviations on to this data and finally by plotting a reconstruction of the data shown as a contour map, including means.  

Show modified version of main plot in same ipynb as the dot histogram. Make contour plot. 

A number of important interpretations were made from this data which directed the subsequent directions of ananlysis. Firslty, this approach was impractically slow owing to the large number of points required to reprepsent the highest intensity readings, even when the number of datapoints was divided by 100. 

Secondly, modelling in 2D did not reproduce reasonably similar clusters to modelling 1D data. This was most apparent when comparing X_voltage data for 3 initial components. Increasing the number of components worsened the model fit, as data from voltages with high intensities were interpreted as a single cluster. Although 3 initial components produced a model reasonably similar to the original data (compare contour plots), using this approach was clearly not viable as the model failed completely at any greater number of componets.

This analysis was repeated using full, diag and a few other params. Nothing made any improvment. See supplemental data. 

The standard deviations were extremely unequal in the x and y directions. 
Try and add in some explanation here as to why this is such a problem for the EM algorithm. 
Therefore one possibility as to why the GMM fit was failing was unequal nature of the data density. Although time and voltage are continuous variables by nature, in these datasets they were effectively being used a discreet data, albeit with many more categories in the time dimension. GMM is suitable for continuous data, so interpoltion of intervening data points, at least in the V dimension and likely in ms as well, was necessary. Of secondary importance the very large range in intensitiy measurements could have been compromising the detection of smaller intensity peaks within the data. 

The contribution of this second point to the GMM fit failure was investigated first as this could be assessed relatively quickly by using some articifical data with a lower dynamic range. The artifical data was generated simply by manually altering voltage sets with n lowest and n highest intensity measurements whilst judging maintenance of the peak shape by eye. At 2 and 3 initial components, lowering the dynamic range did increase sensitivity towards smaller peaks. However, at > 3 initial compoments, the same model failure was observed as in Fig. N. Analyses were therefore concentrated on finding a more computationally reasonable alternative to the representation of intensity measurements and to lessensing the inequality of x and y measurement densities. 

Show Figure N here. 

Pomegranate is a python package that offers a wide range of probabalistic modelling and allows samples to be weighted. This latter point circumvents the requirement for the large number of x-axis points required to represent high intensity measurements and the associated long processing times. It is also possible to include a minimum x and y standard deviation in the model using Pomegranate, which makes it possible to test the hypothesis that the unequal standard deviations in Figure X were the main factor contributing to model failure.

Using the same visualisation scheme as in Figure X, the same datasets (all voltage from hIAPP_A) using the same range of initial components (2 - 6) are modelled as Guassian mixtures using Pomegranate. Minimum standard deviations were initially set using an approximate ratio as that observed in Figure X and then adjusted accordingly. After a few attempts, setting minstdX = 0.5 and minstddevY = 8 was found to yield optimal results, although the had to be determined experimentally for each collection of input files. Pomegranate does not have an inbuilt function for calculating BIC, so this had to be done manually. 

Pom version of Figure X in here.

Although the output model as judged by contour plots could still be improved upon, peaks were clearly no longer being predicted according to voltage group and higher number of initial components was no longer leading to complete failure of the model. The hypothesis that uninterpolated data could not effectively be modelled as Guassian appeared to be correct. The next step was therefore interpolate the data in both voltage and time dimensions. 

There are a variety of python packages for multivariate data interpolation listed at SciPy, of which nearestNDinterpolator, CloughTocher2DInterpolator interp2D(RectBivariateSpline), interpn and RegularGridInterpolator lookedlooked like they might be appropriate. 
Explain a bit why e.g. not linear, grid-like.

Having some familiarity with the scipy package interp1D, I started with interp2D. Interp2D accepts x, y and z data in array-like formats. I performed the interpolation on the human_slice A data using the following code:

The success of the interpolation was initially assessed by way of simple visual comparison with a density plot of the uninterpolated data. The density plot was created using the following code:

Although interp2D appeared to adquantely interpolate the data, fuzzy artefacts were observed at the peripheries of the plot grid. 
Further reading of the documentation prompted the rectate bivariate spline method to be investigated. This required further reshaping of the data into two 1D arrays for X and Y data, and one 2D array for Z data.

Comparison with the density plot suggests the interpolation using RBVspline method had successfully interpolated the data. put in figure. I therefore continued to fit the GMM to the interpolated data using pomegranate. The means and associated errors predicted by the model could be superimposed on a density plot of the interpolated data and the data could be recreated according to the model. 

Need to fix up the 90 degree figure problem here. 

The original interpolated data and recreated data could then be substracted and the residuals plotted. Say what red and blue represents. The disparity between the two suggested that the data interpolation using the RBVspline method hadn't been as succesful as it first seemed. I therefore decided to try another interpolation method, this time the CloughTocher method. 

Explain a bit about this method and why it might be good, why better than the RBVspline method. 

The CloughTocher interpolated accepts data in .... format. The data was reshaped and interpolated using the following code.

As with the RBVspline method, the means and errors were plotted on the interpolated density plot and the residuals of the interpolated data and modelled data were plotted. This time the resiudals appeared much lower, suggesting a considerably better fit. In all three interpolations, 100 data points had been interpolated on the X and Y axes (nsmooth = 100). I therefore settled on the CloughTocher interpolated as the interpolation method of choice. Using an nsmooth of 200 reduced the residuals yet further without coputational time being disporoptionately long. 

Until this point, all method development had been done on data from hIAPP sliceA. Now that a preliminary method had been devleope involding CloughTocher interpolation and Pomegranate, this method had to be tested on hIAPP slice B, rIAPPsliceA and rIAPP sliceB. 

Put in figures

The method appeared highly satifactory in modelling data from all test datasets, as evidenced by the low residuals between input and modelled data. Therefore the next step was to automate optimisation of the number of starting components, quantification of model fit, retrieval of means and variance and quantification of protein unfolding, such that the method could be usefully integrated into some future analytical cIMMS software. 

BIC is **** and therefore will show an appreciable decrease when the optimal number of starting component is chosen. I therefore plotted the change in gradient between BICs caculated from between 2 and 10 components. I also plotted RMSD vs n starting components, to check that this showed a similarly sharp decrease. 

Pomegranate does not have an inbuilt function for caculating the BIC, so this had to be done manually.

Example of code and describe how/why it works. 

The results below indicate that there is general, but not always perfect, agreement between decrease in BIC and RMSD. Optimal n_starting components could be chosen automatically by calculating the number at which the previous gradient is e.g. less than half of the current one. However, given the sometimes inconsistent changes in gradients in the above Fig., and the need to gather results from the full range of starting components in order to work out the best one, the safest policy could be to present the user with all results from all n_components for them to decide. 

The next challenge for a functional user-interface was to optimise results presentation. I considered n things that users would want from the data: a density plot of the original data, the means plus x,y errors of the means fitted to the data, the modelled data with modelled means, visualisation of unfolding profiles over time, visualisation of unfolding profiles over voltage and visualisation of total protein unfolding. 

Of these, the first three have been touched upon already. Visualisation of profiles over both voltage and time could be relatively easily achieved by 'slicing' the 2d-interpolated data at select points along the grid axes. 

Put in relevant code and figures. 

The final result set to present was total unfolding of each protein slice from start to finish time at each of the different voltages. This was not quite as straighforward as it required the starting, or parent, peak had to be defined. This could then be compared to the sum of all the child peaks. The parent peak was initially defined as the modelled mean at the smallest y-axis (voltage) value. However, it became apparent in (rat slice B?) that the first peak along the y-axis might not be the parent peak. The parent peak cannot instead be defined as the most intense peak as some child peaks could be more intense. 

The following code defines the parent peak under a variety of scenarios. Firslty, and most simply, the code assess whether the list position of the y-value for the most intense peak is the same list position of the smallest y-value for the means i.e. if the first mean is also the most intense peak they both should be the first items in np.argsort lists of intensity y-coords and mean y-coords. 

Show relevant code. 

If this is not the case, the code then asks whether the y-value for the maximum intensity peak is less than 15 y-units (i.e. Volts) from the y-value of the first mean. If so, then this peak is taken as the reference peak index. This difference of 15 y-units is based on the smallest difference between a parent peak and the child peak with the next highest y-value in the test datasets on which this study is based. As more test datasets are accumulated, this number can be refined. 

Next few lines of code. 

This effect of this can be demonstrated with the XXXX dataset, where the parent peak is the second peak along the y-axis. Figure X shows datasets XXXX and YYYY. In XXXX the first peak along the y-axis is the most intense and in YYYY the second peak is. Without the 15 y-unit caveat (Fig. Xa), the incorrect parent peak has been chosen for YYYY. Applying the above code chunk corrects this (Fig. Xb). 


If the most intense peak is further away than 15 y-units, this suggests something unusual about the dataset in question, so although the same peak as in Fig. Xb is take as the reference index, the data is flagged with a warning message. 

Complete code chunk. 

In summary, these results describe the process by which a Gaussian mixture model is fitted to generic cIMMS data than varies in time, voltage and intensity, with a view to the process being implemented via a user-interface. The results summary below shows how the output from such software might appear. 

Fig with all graphs, tabulated meand and bics. 