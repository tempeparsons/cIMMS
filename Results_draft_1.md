Results

In this study we applied GMM to data from cIMMS that varied in voltage, time and itensity. The main goal was to establish an automated method by which the number of gaussian peaks within the total data for all voltages for a single slice from the original parent protein species could be determined. This would reveal the decomposition stability of the protein in response to voltages disruption. The mean and variation of decomposition peaks would help determine the positions at which the parent protein was most vulnerable to unfolding. 
<br>

![](./images/fig1_basic_data.png)

<br>

Clustering methods, including GMM, can be applied to one, two or higher dimensional data. In this case, data is measured in time, voltage and intensity. If, however, the intensity dimension is represented by the number of data points at a given time measurement, the dimensionality is reduced. If the clustering is performed separately for each voltage, Gaussian peaks are fitted in a single dimension. The above plot is reproduced with the results from 1-D clustering. 
<br>

![](./images/fig2_basic_w_clustering.png)

<br>
In the above clustering the number of components was estimated by k-means and was set to an initial value of 3, based on visual inspection of the data. Setting the alpha to almost completely transparent means the overlapping datapoints give a limited reflection of the data density. The 1D clustering appears reasonable at 0, 10, 20, 60 and 70 V but without the context of the complete voltage set, clustering at 30 - 50 V appears inaccurate; clearly clustering at higher dimensions is required, so the entirity of the data can be considered. 

The datasets for separate voltages, each containing a number of datapoints proportional to the internsity at each time point, can be combined for modelling in 2D using the same principles for 1D clustering in Scikitlearn shown above. The shape of the data pre-modelling can be shown by combining the 1D datasets shown above, again using transparancy to convey data intensity. The intensity can be shown using either a 2D histogram or a contour plot, for comparison with multi-dimensional modelling scenarios in the next section. 

![](./images/multiple_1D_hists.png)

![](./images/2Dhist_and_contour.png)


The SciKitLearn GMM module contains a number of important parameters requiring optimisation. GMM uses the EM algorithm to find the best model for the data. Essentially, the EM algorithm guesses starting means and variances for a dataset where the means, variances and the number of components (or peaks) means are unknown. The algorithm creates a new dataset with the guessed means and variances, then calculates the probability that the original data came from Gaussian peaks described by the guessed parameters. The contribution of each data point to this total probability score is weighted by each datapoint's distance from the guessed means. This allows the data fit to be improved; if the guessed means were distant from most points, the summed probabiltiy score would be low. The algorithm then alters the guessed parameters so that the assessed probability of all data points is increased. Once the log liklihood of this summed score is not increasing appreciably between guesses (using a pre-determined threshold), then the best data model is said to have been achieved. Clearly, therefore, the method used for taking the initial guess will profoundly impact the final model. 

SciKitLearn offers three main types of ititialisation method: k-means, a random selection from the original data points or a random selection of points close the mean of the total dataset. ###explain k means####. Once the method has been chosen, the number of starting points, must be chosen. Obviously the best number of starting points would be the same as the real number of peak, or components, but this is unknown. 

The optimal number of initial components can be deduced using Bayesian Information Criterion (BIC), for which there is an inbuilt method in SKL. BIC describes how likely a model is given the data, compared to the other models. If the variance in the error between the data and model increases, or if the number of components increases, so too does BIC. The latter point means a large number of initial components is penalised, which can help avoid overfitting unless the model is genuinely complex. If BIC is calculated for a range of components, a steep drop in BIC often indicates the optimal starting number. 

Covariance type is the second important parameter to consider. Covariance describes whether changes in one variable e.g. voltage, impact on another variable e.g arrival time. Our prior knowledge of these biological measurements tells us that in general as proteins are subject to increasing voltage their unfolded-ness, and hence their arrival time, increases. The extent to which these are correlated, i.e. the shape of the covariance, is unknown as this model must be applicable to proteins with varying structural stabilites. Of the four covariance types provided by SciKitLearn ('full', 'tied', 'spherical' and 'diagonal') only 'full' describes an existing but unknown covariance of x and y. 
 
Data for the entire hIAPP_sliceA dataset was arranged as a 2D histogram, as shown in Fig. 3, a 2D GMM could be applied. I investigated the impact on BIC of choosing 2 - 6 initial components using all available initialisation methods and 'full' covariance. Early model-fitting attempts were evaluated visually. Predicted means and standard deviations were plotted on to a 'dot-style' histogram similar to Fig. 3a (Fig. 4a). Different colours for predicted gaussian components were then superimposed (Fig. 4b), a density plot with superimposed component means was used to show the probabilities associated with each datapoint after the EM algorithm had convered (Fig. 4c). To facilitate comparison with the original data in Fig. 3, this probability density was also shown as a contour plot (Fig. 4d). 

![](./images/fig4_skl_gmm_full.png)

A number of important interpretations were made from this data which directed the subsequent directions of analysis. Firstly, this approach was impractically slow owing to the large number of points required to reprepsent the highest intensity readings. This was circumvented by dividing the number of datapoints being divided by 100 although would have reduced resolution in subsequent modelling. 

Secondly, modelling in 2D did not produce even moderately comparable clusters to 1D data models. This was most apparent from 20V and upwards in 4 and 5 inital components, where data from voltages with the highest intensities were interpreted as a single cluster. Comparing contour plots showed that was similar to the original data - but this approach was cleary not viable as it failed completely at n_components > 3. For completeness, this analysis was repeated with 'tied' and 'diag' covariance, an different initialisation methods but none resulted in improved model fits (Fig. 5).

![](./images/fig5_params_completeness.png)

The error bars in Fig. 4a revealed an extremely unequal variance in the x and y directions; probably more than the SKL module would have assumed in any of its covariance parameters. Therefore one possibility as to why the GM fit was failing was the unequal nature of the data density. Although time and voltage are continuous variables by nature, in these datasets they were both effectively being used as discreet data, with NNN categories in the time dimension and up to 8 in the voltage dimension. GMM is suitable for continuous data, so interpoltion of intervening data points in both dimensions was essential. The very large range in intensitiy measurements may have been compromising the detection of smaller intensity peaks in Fig. 4, although this was likely of secondary importance. 

The contribution of this second point to the model failure was investigated first, as this could be assessed relatively quickly by artificially lowering the data range. Global adjustments such as logging or sqaure rooting would have removed the already-Gaussian shape of the data. Instead, artifical data was generated by manually altering voltage sets with the lowest and highest intensity measurements whilst judging maintenance of the peak shape by eye. At 3 initial components, lowering the dynamic range did increase sensitivity towards smaller peaks. An appreciable improvement in fit was observed with either 'full' or 'diag' covariance, or with any initialisation method except 'random' (the discreet nature of the data prevents random sampling of the data space). However, at > 3 initial compoments, the same model failure was observed as in Fig. 4. Analyses were therefore concentrated on data interpolation and finding a faster approach than modelling huge numbers of data points.

![](./images/fig6_fake_data.png)

Pomegranate is a python package that offers a wide range of probabalistic modelling options. It also allows samples to be weighted, which the requirement for the large number of x-axis points required to represent high intensity measurements. Minimum standard deviation can be set in the x and y direction using Pomegranate, meaning the hypothesis that very unequal standard deviations in Figure 4 were the main factor contributing to model failure. 

Using the same visualisation scheme and component numbers as in Figure 4, the hIAPP-sliceA dataset was modelled as Guassian mixtures using Pomegranate. Minimum standard deviations were initially set using an approximate ratio as that observed in Figure 4. After a few attempts, setting minstdX = 0.5 and minstddevY = 8 was found to yield optimal results for this dataset. Pomegranate does not have an inbuilt function for calculating BIC, so this was done manually. 

![](./images/fig7_pom_gmm_full.png)

The results from fitting Gaussians to the data using a defined minimum standard deviation in the Pomegranate module produced very different fits from in Figure 4. A single Gaussian was no longer being explained by high intensity data from a single voltage (Fig. 7b) and the 2D histogram and contour maps of the modelled data (Fig. 7c, d) look much more comparable to those from the original data (Fig. 3), even at greater than 3 components. As the number of components increased and new Gaussians were added, existing one remained in position and credible scenarios of unfolded protein peaks underlying the intensity signal started to materialise (Fig. 7c, d). 

Constraining the minimal standard deviation had showed that the reason for earlier model failures had been the very unequal distrbution of data in the x and y directions, and the Pomegranate module had provided a straightforward means to weight data points according to intensity. However, results in Figure 7 could still not be easily incorporated into software as optimising the minimum standard deviation required extensive manual evaluation. The next step, therefore, was to interpolate the data in both voltage and time dimensions and see whether this allowed Gaussians to be successfully fitted with little or no adjustment of standard deviation settings. Once achieved for the hIAPP_sliceA data, the robustness of this approach could be determined using the three other datasets. 

There are a variety of python packages for multivariate data interpolation listed at SciPy, of which nearestNDinterpolator, CloughTocher2DInterpolator interp2, RectBivariateSpline, interpn and RegularGridInterpolator were appropriate for grid-like data. 
Being familiar with the scipy package interp1D, I decided to start invetigations into optimal 2D data interpolation using Interp2D (although since performing these calcultions this package has been deprecated). Interp2D accepts x(arrival time), y(volatge) and z(intensity) data in array-like formats for non-zero intensity values. Parameters were set to cubic interpolation with fill value of 0. Data was smoothed to 200 points in both x and y dminsions, which proved to be a reasonable balance between smoothing and computational time. A density plot of interpolated data (Fig. 8) was very comparabale to the original data (Fig.3) but low intensity, diffuse artefacts were observed at some peripheries of the data grid (Fig. 8). Neither parameter adjustment, allowing y-values to be interpolated below zero nor clipping grid values at zero could remove these artefacts of interpolation. Although they were unlikely to have altered model fit significantly in the current test datasets, this was nonetheless sub-optimal, so the rect bivariate spline interpolation method was attempted next. The data was reshaped into 1D arrays for X and Y data, and one 2D array for intensity values, this time including zero values, and default parameters were used. 

![](./images/fig8_interp2D.png)

Comparison with all previous density plots (Fig. 3, 7, 8) suggested the interpolation using RBVspline method had been successful (Fig. 9). 

![](./images/fig9_RBVinterpolation.png)

I therefore continued to fit Gaussian Mixtures to the data with Pomegranate. The means and associated errors predicted by the model could be superimposed on a density plot of the interpolated data and the means could be superimposed on the data as recreated. The model initially appeared to be a good fit as the recreated data was visually similar to the original data (Fig. 10). This fit was tested more robustly by plotting the residuals between the original and recreated data, with positive residuals coloured in red and negative ones in blue. 

![](./images/pomegranate_RBVspline.png)

![](./images/RBV_residuals.png)

This plot (Fig. 11a) indicated a good fit between the modelled and original data although, judging by the coloured patches, some small disparities in peak shape remained. I therefore decided to try one more interpolation method, the CloughTocher2DInterpolator, to see if the fit could be improved any further.  

![](./images/Fig11b_CT_residuals.png)

#####Explain a bit about this method and why it might be good, why better than the RBVspline method. ##### Or for the discussion. 

The CloughTocher interpolated accepts a 2D array of x,y data and separate 1D array of z data. The data was reshaped accordingly, and default parameters were used. Residuals in Figure 11b showed a noticeable improvement compared to Figure 11a. Until this point, all method development had been done on data from the hIAPP sliceA dataset. Having established CloughTocher2DOnterpolator as the preferred interpolation method and Pomegrate as the preferred GM modelling module, I tested their performance on the three remaining datasets, hIAPP_sliceB, rIAPP_sliceA and rIAPP_sliceB. For each dataset I plotted interpolated data with estimated gaussian means and variances, data recreated using the modelled gaussian means and variances, and the residuals plus the RMSD between original and modelled data. 
 
####make mega figure#####

The method appeared highly satifactory in modelling data from all test datasets, as evidenced by the plots and low RMSD between input and modelled data. Having established a plausible method which could run reasonably quickly using future software, the next challenge was to consider what parameters it might take and which graphical outputs would be most useful. A very useful outcome of the interpolation was that Pomegranate could be implemented without having to specify a minimum standard deviation for any of the datasets tested, so standard deviation no longer needed to be considered as a parameter. For the majority of the Pomegranate modelling, however, I had been visually inspecting graphical output to estimate the optimal number of components and adjusting this on a trial and error basis, which was obviously not a prcatical approach when desiging software. 

I therefore implemented a method for automating measurement of the BIC and RMSD between original and modelled data for each dataset, on which software users could base their choice of component number (assuming a limit of 10 components).

#############################UP TO HERE#########################

Therefore the next step was to automate optimisation of the number of starting components, quantification of model fit, retrieval of means and variance and quantification of protein unfolding, such that the method could be usefully integrated into some future analytical cIMMS software. 

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
