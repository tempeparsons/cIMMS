<strong>Results</strong>

In this study we applied Gaussian Mixture Modelling (GMM) to cyclic ion mobility mass spectrometry (cIMMS) data from for four test datasets varying in voltage, time and intensity. The main goal was to establish an automated method by which the number of gaussian peaks within the total data for all voltages for a single slice from the original parent protein species could be determined. This would reveal the decomposition stability of the protein in response to voltages disruption. The mean and variation of decomposition peaks would help determine the positions at which the parent protein was most vulnerable to unfolding. 
<br>

![](./images/Fig1.png)

<strong>Figure 1. Detection of ions arriving at the cIMMS detector at select time points for a 'slice' of Islet amyloid polypeptide (hIAPP_A) </strong>. IAPP from either human or rat was partially unfolded during successive rounds in a closed-loop IM separator. The resulting continuum of protein conformations was partitioned into two 'slices' A and B. The structures in each slice were disrupted by increasing voltages, separated once more, and the arrival time of the ensuing ions was measured. Data in Figure 1 shows the intensity of ion arrivals at time points for slice A of human IAPP (hIAPP_A) at each voltage. The sizeable data range is emphasised using shared y-axis values. The three other test datasets (hIAPP_B, rIAPP_A, rIAPP_B) are shown in Supplemental Data (Fig.S1a - c)
<br>
<br>

Clustering methods, including GMM, can be applied to one, two or higher dimensional data. The current data comprises meaurements in time, voltage and intensity. If, however, the intensity dimension is represented by the number of data points at a given time measurement, the dimensionality is reduced. If the clustering is performed separately for each voltage, Gaussian peaks can be fitted in a single dimension. Being unfamiliar with GMM, I began investigations into its applicability to this data by performing 1D clustering on each voltage.  
<br>

![](./images/Fig2.png)

<strong>Figure 2. 1-dimensional Gaussian mixture modelliing of hIAPP_A data.</strong> Data from each voltage shown in Figure 1 was reduced in dimensionality by expressing intensity at the number of points at an arrival time, then subjected to 1D clustering using the sklearn.mixture.GaussianMixture module, with parameters set as follows: n_components=3, max_iter=100, init_params='kmeans', covariance_type='full', random_state=0. Default settings were used for all other paramters and data points were coloured according to final cluster. 
<br>
<br>

In the above clustering Gaussian fittings, the number of components was estimated by k-means and was set to an initial value of 3, based on visual inspection of the data. Setting the alpha to almost completely transparent means the overlapping datapoints give a very rough approximation of the data density. In the hIAPP_A dataset (Fig. 2) and all others (Supplemental Figs.) the 1D clustering appeared reasonable at some voltages but not others (e.g. 30, 40 and 50V). Obviously, it is crucial to consider the individual voltage datasets in the context of the whole protein unfolding event, as not all components will be equally present in each dataset. 

The principal of 2D Gaussian fitting is similar to 1D fitting, except that covariance i.e. how much a change along one axis affects changes along the other axis, must be carefully considered. The longer a protein is subject to a voltage the more it will unfold. However, for any one data point, and therefore any one Gaussian peak, the variation in voltage is not affected by the variation in arrival time.  

Before applying 2D Gaussian fitting, the overall shape of all test datasets was compared to check the likelihood that the same fitting technique would work reasonably well for all datasets. Rather than using datapoints with high transparacies as before, intensity was better visualised using a 2D histogram (Fig. 3A, B, D, E) and a contour plot (Fig. 3F, G, H, I). Overall, there did not seem to be any differences in dataset shape so large as to consider using different fitting methods, although rat data (Fig. 3D, E) looked slightly less gaussian than human data (Fig.3A, B). It was noteworthy that x and y were not the same length in all datasets. Consequently, the modelling code was tested on the same lengthed datasets (hIAPP_A and _B) before the final method written into functions which could account for such variation. 
<br>

![](./images/Fig3AtoD.png)

![](./images/Fig3EtoH.png)

<strong>Figure 3. Different methods for visualising the shape of the data.</strong> Intensity was represented by the number of points at an arrival time, as in Fig.2, and combined (Fig. 3a). This combined data was expressed as a numpy array and plotted as a 2D histogram using the numpy.histogram2d function (Fig. 3b). The array was further plotted as a contour maps using the matplotlib.pyplot.contour function (Fig. 3c). 
<br>
<br>

The SciKitLearn GMM module contains a number of important parameters requiring optimisation. GMM uses the EM algorithm to find the best model for the data. Essentially, the EM algorithm guesses starting means and variances for a dataset where the means, variances and the number of components (or peaks) means are unknown. The algorithm creates a new dataset with the guessed means and variances, then calculates the probability that the original data came from Gaussian peaks described by the guessed parameters. The contribution of each data point to this total probability score is weighted by each datapoint's distance from the guessed means. This allows the data fit to be improved; if the guessed means were distant from most points, the summed probabiltiy score would be low. The algorithm then alters the guessed parameters so that the assessed probability of all data points is increased. Once the log liklihood of this summed score is not increasing appreciably between guesses (using a pre-determined threshold), then the best data model is said to have been achieved. Clearly, therefore, the method used for taking the initial guess will profoundly impact the final model. 

SciKitLearn offers three main types of ititialisation method: k-means, a random selection from the original data points or a random selection of points close the mean of the total dataset. ###explain k means####. Once the method has been chosen, the number of starting points, must be chosen. Obviously the best number of starting points would be the same as the real number of peak, or components, but this is unknown. 

The optimal number of initial components can be deduced using Bayesian Information Criterion (BIC), for which there is an inbuilt method in SKL. BIC describes how likely a model is given the data, compared to the other models. If the variance in the error between the data and model increases, or if the number of components increases, so too does BIC. The latter point means a large number of initial components is penalised, which can help avoid overfitting unless the model is genuinely complex. If BIC is calculated for a range of components, a steep drop in BIC often indicates the optimal starting number. 

Covariance type is the second important parameter to consider. Covariance describes whether changes in one variable e.g. voltage, impact on another variable e.g arrival time. Our prior knowledge of these biological measurements tells us that in general as proteins are subject to increasing voltage their unfolded-ness, and hence their arrival time, increases. The extent to which these are correlated, i.e. the shape of the covariance, is unknown as this model must be applicable to proteins with varying structural stabilites. Of the four covariance types provided by SciKitLearn ('full', 'tied', 'spherical' and 'diagonal') only 'full' describes an existing but unknown covariance of x and y. 
 
Data for the entire hIAPP_sliceA dataset was arranged as a 2D histogram, as shown in Fig. 3, a 2D GMM could be applied. I investigated the impact on BIC of choosing 2 - 6 initial components using all available initialisation methods and 'full' covariance. Early model-fitting attempts were evaluated visually. Predicted means and standard deviations were plotted on to a 'dot-style' histogram similar to Fig. 3a (Fig. 4a). Different colours for predicted gaussian components were then superimposed (Fig. 4b), a density plot with superimposed component means was used to show the probabilities associated with each datapoint after the EM algorithm had convered (Fig. 4c). To facilitate comparison with the original data in Fig. 3, this probability density was also shown as a contour plot (Fig. 4d). 
<br>

![](./images/Fig4.png)

<strong>Figure 4. 2-dimensional Gaussian mixture modelling of hIAPP_A data using the SciKitLearn GMM module.</strong> Data was prepared as described in Fig. 3A, then subjected to 2D clustering using the sklearn.mixture.GaussianMixture module. Between 2 and 5 starting components were tested, initialised using k-means, with covariance type set to 'full'. Default settings were used for all other paramters. In Fig. 4A Gaussian cluster are not coloured so that the means and standard deviations are easily visualised. Next, datapoints are coloured according their assigned peak (Fig. 4b). The data distribution is better visualised using density plots (Fig. 4C) and contour maps (Fig. 4D). Modelled means are shown in red (Figs. 4C, D) and the BIC associated with each number of starting components is given (Fig. 4A). 
<br>
<br>

A number of important interpretations were made from this data which directed the subsequent directions of analysis. Firstly, this approach was impractically slow owing to the large number of points required to reprepsent the highest intensity readings. This was circumvented by dividing the number of datapoints being divided by 100 although would have reduced resolution in subsequent modelling. 

Secondly, modelling in 2D did not produce even moderately comparable clusters to 1D data models. This was most apparent from 20V and upwards in 4 and 5 inital components, where data from voltages with the highest intensities were interpreted as a single cluster. Comparing contour plots showed that was similar to the original data - but this approach was cleary not viable as it failed completely at n_components > 3. For completeness, this analysis was repeated with 'tied' and 'diag' covariance, an different initialisation methods but none resulted in improved model fits (Fig. 5).
<br>

![](./images/Fig5.png)

<strong>Figure 5. Impact of parameter adjustment on fitting Gaussian peaks to hIAPP_A data, using the SciKitLearn GMM module.</strong> Gaussian mixtures were modelled using data prepared as for Fig. 4 but, using a 3 starting components and a set random_state, all possible initialisation methods and covariance types were investigated. Only results for covariance type 'full' and 'diag' are shown here. Parameters are detailed in subplot titles. Clusters are coloured using matplotlib colormap 'viridis'. 
<br>
<br>

The error bars in Fig. 4a revealed an extremely unequal variance in the x and y directions; probably more than the SKL module would have assumed in any of its covariance parameters. Therefore one possibility as to why the GM fit was failing was the unequal nature of the data density. Although time and voltage are continuous variables by nature, in these datasets they were both effectively being used as discreet data, with NNN categories in the time dimension and up to 8 in the voltage dimension. GMM is suitable for continuous data, so interpoltion of intervening data points in both dimensions was essential. The very large range in intensitiy measurements may have been compromising the detection of smaller intensity peaks in Fig. 4, although this was likely of secondary importance. 

The contribution of this second point to the model failure was investigated first, as this could be assessed relatively quickly by artificially lowering the data range. Global adjustments such as logging or sqaure rooting would have removed the Gaussian shape of the data. Instead, artifical data was generated by manually altering voltage sets with the lowest and highest intensity measurements whilst judging maintenance of the peak shape by eye. For ease of comparison, the same analysis was performed as in Figure 4. At 2, 3 and 4 components, the the best fitting model using the artificial data was much closer to the original shape of the real data than any model fitted to the real data (Fig. 6). This suggested that the very large data range was compromising the ability of the SciKitLearn GMM function to fit Gaussians to this data. However at 4, and especially 5 components, the same effect was observed as in Figure 4 where the function started to define cluster by voltage rather than arrival times of unfolding proteins. This indicated that fitting failure observed in Figure 4 was only partially due to the large data range; the extremely unequal data density in X and Y directions was probably of greater importance. 
<br>

![](./images/Fig6.png)

<strong>Figure 6. 2-dimensional Gaussian mixture modelling of semi-artifical hIAPP_A data using the sklearn.mixture.GaussianMixture module.</strong> Source data differed from that shown in Fig. 4 in that files containing the most intense peaks (10 Volts) least intense peaks (60 and 70 Volts) had been artificially lowered and raised, respectively. Line plots of artifical data are shown in Supplemental Data. All other parameters were set as in Figure 4. 
<br>
<br>

However a third, and more immediate, problem was that as the intensity measurement was being represented by the number of datapoints at an arrival time, the datasets were enormous and investigating more than a single number of starting components was becoming unfeasible. The need to represent intensity this way can be eliminted by using a different modelling module. 

Pomegranate is a python package that offers a wide range of probabalistic modelling options. It also allows data points to be weighted, making it easier to deal with high intensity measurements. Required data inputs are a grid of coordinates for each datapoint in one array and another array containing the weighting of each point. 'Full' covariance is achieved by using the MultivariateGaussianDistribution function from Pomegranate's GeneralMixtureModel. This is converted to the SciKitLearn equivalent of 'diag' i.e. independent x and y, by defining the minimum standard x and y deviation as classes and using these in place of MultivariateGaussianDistribution. The minimum standard deviation in these classes can, of course, be changed - thereby allowing the hypothesis to be tested that very unequal standard deviations in Figure 4 were the principal factor behind model failure.

Using the same visualisation scheme and component numbers as in Figures 4 and 6, the hIAPP-sliceA dataset was modelled as Guassian mixtures using Pomegranate, with each datapoint weighted according to its intensity. Co-ordinates for zero-intensity data points were removed from the data grid so as these did not contribute to the model fit (this was verified, as shown in Supplemental Data) and this made results visually comparable to previous figures. Firstly, I attempted to reproduce the results from SciKitLearn in which the fit had failed by using Pomegranate's MultivariateGaussianDistribution function. This indeed did produce a similarly failing fit (Figure SX), indicating the two methods were fundamentally similar. Setting minimum standard deviation classes dramatically improved the model; Gaussians now followed intensities rather than voltage at >2 components and the underlying shape of the data was not changing as components were added. Standard deviations were initially set using an approximate ratio as that observed in Figure 4. After a few attempts, setting minstdX = 0.5 and minstddevY = 8 was found to yield optimal results for the hIAPP_A dataset. BIC is absent from intial investigation using Pomegranate, as this is not calculated automatically as in SciKitLean. BIC calculations in Pomegranate are described in subsequent paragraphs. 
<br>

![](./images/Fig7.png)

<strong>Figure 7. 2-dimensional Gaussian mixture modelling of hIAPP_A data using the Pomegranate GeneralMixtureModel module.</strong> Subplot descriptions are as for Figures 4 and 6.  
<br>
<br>

Constraining the minimal standard deviation had showed that the reason for earlier model failures had been the very unequal distrbution of data in the x and y directions, and the Pomegranate module had provided a straightforward means to weight data points according to intensity. However, results in Figure 7 could still not be easily incorporated into software as optimising the minimum standard deviation required extensive manual evaluation for each dataset. The next step was to interpolate the data in both voltage and time dimensions and see whether this allowed Gaussians to be successfully fitted with little or no adjustment of standard deviation settings. Once achieved for the hIAPP_sliceA data, the robustness of this approach could be determined using the three other datasets. 

There are a variety of python packages for multivariate data interpolation listed at SciPy, of which nearestNDinterpolator, CloughTocher2DInterpolator interp2, RectBivariateSpline, interpn and RegularGridInterpolator were appropriate for grid-like data. 
Being familiar with the scipy package interp1D, I decided to start invetigations into optimal 2D data interpolation using Interp2D (although since performing these calcultions this package has been deprecated). Interp2D accepts x(arrival time), y(volatge) and z(intensity) data in array-like formats for non-zero intensity values. Parameters were set to cubic interpolation with fill value of 0. Data was smoothed to 200 points in both x and y dminsions, which proved to be a reasonable balance between smoothing and computational time. A density plot of interpolated data (Fig. 8) was very comparabale to the original data (Fig.3) but low intensity, diffuse artefacts were observed at some peripheries of the data grid (Fig. 8). Neither parameter adjustment, allowing y-values to be interpolated below zero nor clipping grid values at zero could remove these artefacts of interpolation. Although they were unlikely to have altered model fit significantly in the current test datasets, this was nonetheless sub-optimal, so the rect bivariate spline interpolation method was attempted next. The data was reshaped into 1D arrays for X and Y data, and one 2D array for intensity values, this time including zero values, and default parameters were used. 
<br>

![](./images/Fig8.png)

<strong>Figure 8. Interpolation of arrival time, voltage and intensity data in the hIAPP_A dataset using Interp2D and RectBivariateSpline interpolation.</strong> In Figure 8A, data from each dimension was formatted as an array and interpolated using Interp2D with the following parameters altered from default: kind = 'cubic', fill value = 0.0. In Figure 8B data was interpolated using the RectBivariateSpline function with parameters were set as: bbox=[None, None, None, None], kx=3, ky=3, s=0. All other parameters ere left as default. In both Figure 8A and B, 200 data points were interpolated between 55 and 80 ms and between -10 and 70V in the time and voltage dimensions, respectively. 
<br>
<br>
Comparison of the RectBivariateSplie interpolation (Fig. 8B) with the original data (Fig. 3) suggested the interpolation using RBVspline method had been successful (Fig. 9). I therefore continued to fit Gaussian Mixtures to the data with Pomegranate, visualising the original data (Fig. 9A), modelled data (Fig. 9B), means and standard deviations as in previous figures. The model initially appeared to be a good fit as the modelled data was visually similar to the original data (Fig. 9A, B). This fit was tested more robustly by plotting the residuals between the original and recreated data, with positive residuals coloured in red and negative ones in blue (Fig. 9C). 
<br>

![](./images/Fig9.png)

<strong>Figure 9. Interpolation of hIAPP_A data using the RectBivariateSpline method from SciPiy.</strong> Figure 9A shows a density plot of the original data. Fitted means and standard deviations are overlayed on data fitted using Pomegrate (Fig. 9B). The original and modelled data were normalised, subtracted and the residuals plotted, using a hot-cool color map to indicate postive-negative values, along with the RMSD value (Fig. 9C). 
<br>
<br>
The residual plot indicated a reasonable fit between the modelled and original data although, judging by the coloured patches, some small disparities in peak shape remained. I therefore decided to try one more interpolation method, the CloughTocher2DInterpolator, to see if the fit could be improved any further.  

#####Explain a bit about this method and why it might be good, why slightly better than the RBVspline method. ##### Maybe for the discussion. 

The CloughTocher interpolated accepts a 2D array of x,y data and separate 1D array of z data. The data was reshaped accordingly, and default parameters were used. Residuals in Figure 10C showed a small but noticeable improvement compared to Figure 9C (RMSD x vs x). 

![](./images/Fig10.png)

<strong>Figure 10. Interpolation of hIAPP_A data using the CloughTocher interpolation method from SciPy.</strong> Figures 10A, B and C are as for Figure 9, excepting the interpolation method. 
<br>
<br>
<br>
Given that the improvement in fit was not huge, I tested the performance of each interpolation method on the three remaining datasets, hIAPP_sliceB, rIAPP_sliceA and rIAPP_sliceB. 
<br>
<br>

![](./images/Fig11.png)

<strong>Figure 11. Interpolation of hIAPP_B data using the CloughTocher interpolation method from SciPy.</strong> In all cases, the number of components was optimised using data from Figure 14. 
<br>
<br>
<br>

![](./images/Fig12.png)

<strong>Figure 12. Interpolation of hIAPP_B data using the CloughTocher interpolation method from SciPy.</strong>
<br>
<br>
<br>

![](./images/Fig13.png)

<strong>Figures 11. Interpolation of hIAPP_B data using the CloughTocher interpolation method from SciPy.</strong>
<br>
<br>
<br>
The CloughTocher interpolater consistently performed better than the RectBivariate Spline interpolation method, judging by RMSD of original vs. modelled data (Figs. 10 - 13), so this became the method of choice. Having established a plausible method which could run reasonably quickly using future software, the next challenge was to consider what parameters it might take and which graphical outputs would be most useful. A very useful outcome of the interpolation was that Pomegranate could be implemented without having to specify a minimum standard deviation for any of the datasets tested, so standard deviation no longer needed to be considered as a parameter. For the majority of the Pomegranate modelling, however, I had been visually inspecting graphical output to estimate the optimal number of components and adjusting this on a trial and error basis, which is not a possible approach for automated software. 

I therefore implemented a method for automating measurement of the BIC and RMSD between original and modelled data for each dataset, on which software users could base their choice of component number (assuming a limit of 10 components). As mentioned, Pomegranate does not have an inbuilt method for calculating BIC. BIC has been introduced briefly already but must be now be expained in more detailed. BIC is defined as: BIC = k ln(n) - 2 ln(L) where L is "the maximized value of the likelihood function of the model" [ref wikipedia], or the probability of the data having been generated by the best fitting model, k is the number of components and n is the number of data points being fitted in the model. In log space the goodness of model fit is therefore directly proportional to the number of components and size of the dataset. As L increases, so the value of BIC decreases unless k is large - hence how the BIC value is penalised by a large number of components. This is calculated in line XXXX of code chunk XXXX: bic = float(df * np.log(len(smoothxycoords)) - 2.0 * lp). The relative decrease in BIC was emphasised by plotting the change in gradient between n components, rather than the BIC itself (Fig. 13). 

![](./images/Fig14.png)

<strong>Figure 14. Assessment of optimal number of components for each of the four test datasets using BIC and RMSD between original and modelled data.</strong> Figures 14A - D show the BIC calculated as described above from the Pomegranate output. Up to 10 components were tested for each dataset. Figures 14E - H summarise the result of the RMSD scores show in Figures 9 - 13. For all plots, error bars show standard error for the mean of 5 repeats.
<br>
<br>
Results in Figure 14 show general, but not completely consistent, agreement between BIC and RMSD. A scenario in which choice of components was fully automated might involve, for example, calculating the number of compontents at which the previous gradient was less than half the current one. However, given the sometimes inconsistent results, the safest policy may be to present the user with all the data in Figure 13 for them to decide the optimal number of components.  

An essential part of a functional user-interface is data visualistion. I thought that users would need to see a density plot of their input data, before and after interpolation, the component means and x,y standrd deviations, a denisty plot of modelled data with means superimposed, interpolated profiles at user-selected time points, iterpolated profiles and select volatages, and an assessment of overall protein stability throughout the experiment. 

Density plots, means and standard devatiations have been explored in previous figures. Intensity profiles were visualised over voltage and time by 'slicing' data across and along the grid axes (Fig. 15). 

![](./images/Fig15.png)

<strong>Figure 15. Slices horizontally (Fig. 15A) or vertically (Fig. 15B) across the data grid of the CloughTocher-interpolated hIAPP_A dataset prior to Gaussian fitting.</strong> Fig. 15A shows horizontal cross sections from the start to halfway up the Y-axis of the density plot shown in Fig. 10. Each line represents intensities at a cross section in increments of 10. Fig. 15B shows vertical cross sections of the data grid along the X-axis. Data was sectioned over columns equivalent to 55 to 67 mins in Fig. 10, in increments of 5.
<br>
<br>  

Finlly, I decided to present an overall view of protein stability by comparing the proportion of the initial protein peak to all unfolding/unfolded peaks as the voltage was increased. This required that an initial parent peak, present at 0V, was correctly defined and that the area under this peak was measured separately from all the child peaks for all the voltages. Conversely, the sum area under all the child peaks had to be measured at all volatges. Although conceptually straightforward, robust identification of the parent peak is critical in this approach. An obvious first step would be to define the parent peak as the first along the Y(Volts)-axis and X(time)-axis.

![](./images/Fig16.png)
![](./images/Fig18.png)

<strong>Figure 16. Density and line plots showing the intensities of the parent peak compared to child peaks.</strong>. This figure shows plots for hIAPP_A and rIAPP_A, in which identification fo the parent peak was straightforward as it was both the first and the most intense peak in the y-direction. All Gaussians identified in the fitted data and their means are shown in Fig. 16A, E. The child (Fig. 16B, F) and parent (Fig. 16C, G) peaks are visualised separately. In the absence of the parent peak, the color map is re-scaled to the intensity of the child peaks (Fig. B, F). Finally, the relative intensities of parent and child peaks are shown (Fig.D, H). 
<br>
<br>

Although this worked fine in datasets where the first peak was the most intense peak, such as rIAPP_sliceA and hIAPP_sliceA, in rIAPP_sliceB low-intensity peaks were detected at a lower voltage and/or time than the parent peak. The more complex the starting sample, the more plausible such a scenario becomes. Therefore the code was adapted to check whether the first peak along the Y-axis was the most intense by comparing positions in np.argsort lists of mean intensities and mean y-coordinates. When this was not the case, the code then asks whether the y-coordinates for the highest intensity mean is less than 12 y-unit away from the first peak. If so, this more intense peak is accepted as the parent peak. The threshold of 12 y-units is based on the voltage required to produce a substantial second peak in these four test datasets. It is difficult to define a more sophisticated threshold based on only four test datasets, although one can be envisaged which uses both X and Y coordinates based on a radius around the parent peak from e.g. 20 datasets. 

![](./images/Fig19.png)
<strong>Figure 17. Identification of parent and child peaks in the rIAPP_B dataset where a less intense peak, close to the genuine parent peak, occurs first in the y-direction.</strong> Fig. 17A, B and C ddescriptions are as for Fig. 16A, B and C. 

In the current implemntation of the code, if the most intense peak is more than 12 y-units away from the first peak on the Y-axis (as is the case in hIAPP_sliceB), this peak is still chosen as the parent peak but a warning is triggered advising the user to examine the results manually. Again, with more test datasets, this trigger-threshold could be much better defined. 

![](./images/Fig17.png)
<strong>Figure 18. Identification of parent and child peaks in the hIAPP_B dataset where identification of the parent peak is not straightforward, so a warning message is triggered.</strong> Fig. 18A, B and C descriptions are as for Fig. 16A, B and C. 



Taken together, these results show that Gaussian Mixture Modelling provides a mechanism for deconvoluting the signal from circular ion mobility mass spectrometry, although the large data range means than modules enabling weighting of datapints, such as Pomegranate, are more sutabel than more commonly used modeules such as SciKitLearn. Sucessfull Gaussian modelling is subject to data interpolation inbetween the selected time points and voltages, otherwise the data is not continuous and cannnot be modelled using this method. As evidenced by these results, the interpolation method used can have an appeciable effect on how well Gaussians can be fitted to the data. These results also start to show how data input, processing and results output could be incorporated into software for modelling circular ion mobility data. 

