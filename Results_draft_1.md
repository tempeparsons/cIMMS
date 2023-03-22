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

Comparison with all previous density plots (Fig. 3, 7, 8) suggested the interpolation using RBVspline method had been successful (Fig. 9). I therefore continued to fit Gaussian Mixtures to the data with Pomegranate. The means and associated errors predicted by the model could be superimposed on a density plot of the interpolated data and the means could be superimposed on the data as recreated. The model initially appeared to be a good fit as the recreated data was visually similar to the original data (Fig. 10). This fit was tested more robustly by plotting the residuals between the original and recreated data, with positive residuals coloured in red and negative ones in blue. 


![](./images/RB_GMM_HA.png) #swap this for human when you have it

This plot (Fig. 11a) indicated a reasonable fit between the modelled and original data although, judging by the coloured patches, some small disparities in peak shape remained. I therefore decided to try one more interpolation method, the CloughTocher2DInterpolator, to see if the fit could be improved any further.  

#####Explain a bit about this method and why it might be good, why better than the RBVspline method. ##### Maybe for the discussion. 

The CloughTocher interpolated accepts a 2D array of x,y data and separate 1D array of z data. The data was reshaped accordingly, and default parameters were used. Residuals in Figure 11b showed a small but noticeable improvement compared to Figure 11a (RMSD x vs x). 

![](./images/CT_GMM_HA.png)

Up until this point I had been developing the method exclusively on the Human IAPP sliceA dataset. As the improvement in fit using the two different interpolation methods was only marginal, I tested the performance on the two methods on the three remaining datasets, hIAPP_sliceB, rIAPP_sliceA and rIAPP_sliceB. 

![](./images/RB_GMM_HB.png)
![](./images/CT_GMM_HB.png)

![](./images/RB_GMM_RA.png)
![](./images/CT_GMM_RA.png)

![](./images/RB_GMM_RB.png)
![](./images/CT_GMM_RB.png)

The CloughTocher interpolater consistently performed about twice as well as the RectBivariate Spline interpolation method, judging by RMSD of original vs. modelled data (Table 1), so this became the method of choice. Having established a plausible method which could run reasonably quickly using future software, the next challenge was to consider what parameters it might take and which graphical outputs would be most useful. A very useful outcome of the interpolation was that Pomegranate could be implemented without having to specify a minimum standard deviation for any of the datasets tested, so standard deviation no longer needed to be considered as a parameter. For the majority of the Pomegranate modelling, however, I had been visually inspecting graphical output to estimate the optimal number of components and adjusting this on a trial and error basis, which was obviously not a prcatical approach when desiging software. 

I therefore implemented a method for automating measurement of the BIC and RMSD between original and modelled data for each dataset, on which software users could base their choice of component number (assuming a limit of 10 components). As mentioned, Pomegranate does not have an inbuilt method for calculating BIC. BIC has been introduced briefly already but must be now be expained in more detailed. BIC is defined as: BIC = k ln(n) - 2 ln(L) where L is "the maximized value of the likelihood function of the model", or the probability of the data having been generated by the best fitting model, k is the number of components and n is the number of data points being fitted in the model. In log space the goodness of model fit is therefore directly proportional to the number of components and size of the dataset. As L increases, so the value of BIC decreases unless k is large - hence how the BIC value is penalised by a large number of components. This is calculated in line XXXX: bic = float(df * np.log(len(smoothxycoords)) - 2.0 * lp). The relative decrease in BIC was emphasised by plotting the change in gradient between n components, rather than the BIC itself (Fig. 13). 

![](./images/bic_getter_HB.png)
![](./images/bic_getter_RA.png)
![](./images/bic_getter_RB.png)

Results in Figure 13 show general, but not completely consistent, agreement between BIC and RMSD. A scenario in which choice of components was fully automated might involve, for example, calculating the number of compontents at which the previous gradient was less than half the current one. However, given the sometimes inconsistent results, the safest policy may be to present the user with all the data in Figure 13 for them to decide the optimal number of components.  

An essential part of a functional user-interface is data visualistion. I thought that users would need to see a density plot of their input data, before and after interpolation, the component means and x,y standrd deviations, a denisty plot of modelled data with means superimposed, interpolated profiles at user-selected time points, iterpolated profiles and select volatages, and an assessment of overall protein stability throughout the experiment. 

Density plots, means and standard devatiations have already been explored. For visualisation of profiles over voltage and time, data was 'sliced' along the grid axes, as in Figure 14. 

![](./images/slices_HA.png)


Finlly, I decided to present an overall view of protein stability by comparing the proportion of the initil protein peak to all unfolding/unfolded peaks as the voltage was increased. This required that an initial parent peak, present at 0V, was correctly defined and that the area under this peak was measured separately from all the child peaks for all the voltages. Conversely, the sum area under all the child peaks had to be measured at all volatges. Although conceptually straightforward, robust identification of the parent peak is critical in this approach. An obvious first step would be to define the parent peak as the first along the Y(Volts)-axis and X(time)-axis. Although this worked fine in datasets where the first peak was the most intense peak, such as rIAPP_sliceA, in hIAPP_sliceA and rIAPP_sliceB low-intensity peaks were detected at a lower voltage and/or time than the parent peak. The more complex the starting sample, the more plausible such a scenario becomes. Therefore the code was adapted to check whether the first peak along the Y-axis was the most intense by comparing positions in np.argsort lists of mean intensities and mean y-coordinates. When this was not the case, the code then asks whether the y-coordinates for the highest intensity mean is less than 12 y-unit away from the first peak. If so, this more intense peak is accepted as the parent peak. The threshold of 12 y-units is based on the voltage required to produce a substantial second peak in these four test datasets. It is difficult to define a more sophisticated threshold based on only four test datasets, although one can be envisaged which uses both X and Y coordinates based on a radius around the parent peak from e.g. 20 datasets. In the current implemntation of the code, if the most intense peak is more than 12 y-units away from the first peak on the Y-axis (as is the case in hIAPP_sliceB), this peak is still chosen as the parent peak but a warning is triggered advising the user to examine the results manually. Again, with more test datasets, this trigger-threshold could be much better defined.   

![](./images/parent_child_HA.png)

In summary, these results describe the process by which a Gaussian mixture model is fitted to generic cIMMS data than varies in time, voltage and intensity, with a view to the process being implemented via a user-interface. The results summary below shows how the output from such software might appear. 

