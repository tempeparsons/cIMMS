<strong>Results</strong>

In this study we applied Gaussian Mixture Modelling (GMM) to cyclic ion mobility mass spectrometry (cIMMS) data from for four test datasets. Each dataset described the behaviour of a partially unfolded structures of a peptide hormone after it was exposed to increasing voltages that further disrupted its folding. From each of two types of islet amyloid polypeptide peptide hormone (human; hIAPP, rat;rIAPP), two partially unfolded structures had been sampled (A and B), making four datasets in total (hIAPP_A, hIAPP_B, rIAPP_A, rIAPP_B). As these sampled structures were subjectd to inceasing voltages, they unfolded yet further. The presence of a structure is marked by a sharp peak in ion intensity at the detector. More unfolded structures take longer to arrive at the detector. However, the parameters of each peak (and therefore the nature of the structure) are not immediately obvious from the entire set of arrival time data for all tested voltages (Figure 1), although these peaks can be assumed to have Gaussian distributions. The results below outline a method by which the number of peaks and their parameters can be determined from total arrival time data. 
<br>
Clustering methods, including GMM, can be applied to one, two or higher dimensional data. The current data comprises meaurements in time, voltage and intensity. If, however, intensity is represented by the frequency of data points at a given time measurement, the data effectively takes on the properties of a histogram and the dimensionality is reduced. If the clustering is performed separately for each voltage, Gaussian peaks can be fitted in a single dimension. Being unfamiliar with GMM, I began investigations into its applicability to this data by performing 1D clustering on each voltage using the GMM module in the SciKitLearn library (Figure 1).  
<br>

![](./images/Fig2.png)

<strong>Figure 1. 1-dimensional Gaussian mixture modelliing of hIAPP_A data.</strong> Data from each voltage shown in Figure 1 was reduced in dimensionality by expressing intensity at the number of points at an arrival time, then subjected to 1D clustering using the sklearn.mixture.GaussianMixture module. The following parameters were applied to all voltages: n_components=4, max_iter=100, init_params='kmeans', covariance_type='full', random_state=0. Default settings were used for all other paramters and data points were coloured by whichever component the original data point was fitted to. The relationship between fitted peaks and the total arrival time spectra is shown by superimposing line plots of arrival time and intensity for each voltage. Note the high intensity range on the y-axes of subplots.
<br>
<br>
In the above clustering Gaussian fittings, the number of components was estimated by k-means and was set to an initial value of 4 for all voltages. Figure 1 shows how when Gaussians are fitted to each voltage in isolation, more components get assigned to high-intensity areas (e.g. area at ~63.5 ms), despite only a single component being assinged to the same region when its intensity was lower. Different voltages clearly contain different number of components and a component may originate at one voltage, with its apex in another. Figure 1 shows that datasets are not appropriate for fitting in one dimension; fitting in 2D, with all voltages considered together, is essential. 

The principal of 2D Gaussian fitting is similar to 1D fitting, except that covariance i.e. how much a change along one axis affects change along the other axis, must be carefully considered. The longer a protein is subject to a voltage the more it will unfold. However, for any one data point, and therefore any one Gaussian peak, the variation in voltage is independent of the variation in arrival time.  

Before applying 2D Gaussian fitting, the overall shape of all test datasets was compared to check the likelihood that the same fitting technique would work reasonably well for all datasets. Intensity was better visualised using a 2D histogram (Fig. 2A, B, D, E) and a contour plot (Fig. 2F, G, H, I), with intensities represented by a heatmap. Overall, there did not seem to be any differences in dataset shape so large as to consider using different fitting methods, although rat data (Fig. 2D, E) looked slightly less gaussian than human data (Fig.2A, B). It was noteworthy that x and y were not the same length in all datasets. Consequently, the modelling code was tested on the same lengthed datasets (hIAPP_A and _B) before the final methods were written into functions which could account for this. 

![](./images/Fig3AtoD.png)

![](./images/Fig3EtoH.png)

<strong>Figure 3. Visualising the shapes of a dataset using 2d histograms and contour maps.</strong> 2D arrays were generated with one layer containing non-zero arrival times proportional to the intensity and another with corresponding voltages. For each dataset this was plotted as a 2D histogram using the numpy.histogram2d function and coloured from highest intensity (yellow) to lowest (black) as follows: hIAPP_A; 2A, hIAPP_B; 2B, rIAPP_A; 2C, rIAPP_B; 2D. Equivalent arrays were generated for plotting contour maps (Figs. 2E - H). 
<br>
<br>

A number of parameters in the SciKitLearn GMM modeule mst be optimised for fitting 2-dimensional data. GMM uses the EM algorithm to find the best model for the data. Essentially, the EM algorithm guesses starting means and variances for a dataset where the means, variances and the number of components (or peaks) are unknown. The algorithm creates a new dataset with the guessed means and variances, then calculates the probability that the original data came from Gaussian peaks described by the guessed parameters. The contribution of each data point to this total probability score is weighted by each datapoint's distance from the guessed means. This allows the data fit to be improved; if the guessed means were distant from most points, the summed probabiltiy score would be low. The algorithm then alters the guessed parameters so that the assessed probability of all data points is increased. Once the log liklihood of this summed score is not increasing appreciably between guesses (using a pre-determined threshold), then the best data model is said to have been achieved. Clearly, therefore, the method used for taking the initial guess will profoundly impact the final model. 

SciKitLearn offers three main types of ititialisation method: k-means, a random selection from the original data points or a random selection of points close the mean of the total dataset. Once the method has been chosen, the number of components must be chosen. Ideally this would be the same as the real number of Gaussian peaks but this is, of course, unknown. 

The optimal number of components can be deduced using Bayesian Information Criterion (BIC), for which there is an inbuilt method in SKL. BIC describes how likely a model is given the data, compared to the other models. If the variance in the error between the data and model increases, or if the number of components increases, so too does BIC. The latter point means a large number of initial components is penalised, which can help avoid overfitting unless the model is genuinely complex. If BIC is calculated for a range of components, a steep drop in BIC often indicates the optimal number. 

Covariance type is another parameter to consider. Covariance describes whether changes in one variable e.g. voltage, impact on another variable e.g arrival time. As mentioned, these are expected to be independent. Of the four covariance types provided by SciKitLearn ('full', 'tied', 'spherical' and 'diagonal') 'diagonal' describes indepenent covariance of x and y - although all convariance types were eventually examined. 
 
Data was arranged as a 2D numpy array, as descried for Figure 2, and 2D GMM was applied. BIC was ascertained for 2 - 5 components, otherwise model fitting was evaluated visually, using coloured datapoint plots similar to to Figure 1, density and contour plots. 

![](./images/Fig4.png)

<strong>Figure 3. 2-dimensional Gaussian mixture modelling of hIAPP_A data using the SciKitLearn GMM module.</strong> Between 2 and 5 components were tested, initialised using k-means, with covariance type set to 'diag'. Default settings were used for all other paramters. In Fig. 3A Gaussian cluster are not coloured so that the means and standard deviations are easily visualised. Datapoints from the original data are coloured according their assigned Gaussian component(3B). Fitted data distribution is visualised using density plots (3C) and contour maps (3D). Modelled means are shown in red (3C, D) and the BIC associated with each components number is given in the plot titles (3A). 
<br>
<br>
A number of important interpretations were made from this data which directed the subsequent directions of analysis. Firstly, despite the intensity values being divided by 100, this approach was impractically slow owing to the large number of points required to reprepsent the highest intensity readings. 

Secondly, modelling in 2D did not produce even moderately comparable clusters (Fig. 3E-H) to 1D data models (Fig. 1). This was most apparent when using > 3 components (Fig. 3F-H), where components became fitted to voltage datasets rather than the Gaussians spanning them. Consequently the overall data shape (Fig. 3I-P) only ressembled the original data when 2 components (Fig. 3I, M) were used.

For completeness, different covariance and initialisation parameters were investigated but none resulted in improved model fits (Fig. 4).
<br>

![](./images/Fig5.png)

<strong>Figure 4. Impact of parameter adjustment on fitting Gaussian peaks to hIAPP_A data, using the SciKitLearn GMM module.</strong> Gaussian mixtures were modelled using data prepared as for Fig.34 but, using a 3 starting components and a set random_state, all possible initialisation methods and covariance types were investigated. Only results for covariance type 'full' and 'diag' are shown here. Parameters are detailed in subplot titles. Clusters are coloured using matplotlib colormap 'viridis' as in Figure 1. 
<br>
<br>

The error bars in Figure 3A-D revealed an extremely unequal variance in the x and y directions. If more unequal that permitted by the 'diag' covariance type, this could have explained the failure to fit the data. Although time and voltage are continuous variables by nature, in these datasets they were both effectively being used as discreet data, with 200 categories in the time dimension and up to 8 in the voltage dimension. GMM is suitable for continuous data, so interpoltion of intervening data points in both dimensions was going to be necessary. The very large range in intensitiy measurements at may have been compromising the detection of smaller intensity peaks in Fig. 3, although this was likely of secondary importance. 

The contribution of this second point to the model failure was investigated first, as this could be assessed relatively quickly by artificially lowering the data range. Global adjustments such as logging or sqaure rooting would have removed the Gaussian shape of the data. Instead, artifical data was generated by manually altering voltage sets with the lowest and highest intensity measurements whilst judging maintenance of the peak shape by eye. For ease of comparison, the same analysis was performed as in Figure 3. At 2, 3 and 4 components, the the best fitting model using the artificial data was closer to the original shape of the real data than any model fitted to the real data (Fig. 5). This suggested that the very large data range was compromising the ability of the SciKitLearn GMM function to fit Gaussians to this data. However at 4, and especially 5 components, the same effect was observed as in Figure 3 where the function started to define cluster by voltage rather than arrival times of unfolding proteins; the very large data range likley had onyl marginal impact on model fitting failure. 
<br>

![](./images/Fig6.png)

<strong>Figure 5. 2-dimensional Gaussian mixture modelling of semi-artifical hIAPP_A data using the SKL GMM module.</strong> Source data differed from that shown in Fig. 3 in that files containing the most intense peaks (10 Volts) least intense peaks (60 and 70 Volts) had been artificially lowered and raised, respectively. Otherwise all other parameters and analyses were set as in Figure 3. 
<br>
<br>
However a third, more immediate, problem was that the number of datapoints at representing the greatest intensities meant datasets were enormous and investigating multiple components was becoming unfeasible. The need to represent intensity this way was eliminted by using a different modelling library. Pomegranate is a python package that offers a wide range of probabalistic modelling options. It also allows data points to be weighted, making it easier to deal with high intensity measurements. Required data inputs are a grid of coordinates for each datapoint in one array and another array containing the weighting of each point. The MultivariateGaussianDistribution function from Pomegranate's GeneralMixtureModel module uses 'full' convariance by default. This is converted to independent x and y, by defining the minimum standard x and y deviation as classes and using these in place of MultivariateGaussianDistribution. The minimum standard deviation in these classes can then be changed - thereby allowing the hypothesis to be tested that very unequal standard deviations in Figure 3 were the principal factor behind model failure.

Using the same visualisation scheme and component numbers as in Figures 3 and 5, Gaussians were fitted to the hIAPP-sliceA dataset was using Pomegranate, with datapoints weighted by intensity. Firstly, I attempted to reproduce the results from SciKitLearn in which the fit had failed by using Pomegranate's MultivariateGaussianDistribution function. This indeed did produce a similarly failing fit, indicating the two methods were fundamentally similar. Setting minimum standard deviation classes dramatically improved the model; Gaussians now followed intensities rather than voltage at >2 components and the underlying shape of the data was not changing as components were added. Standard deviations were initially set using an approximate ratio as that observed in Figure 4. After a few attempts, setting minstdX = 0.5 and minstddevY = 8 was found to yield optimal results for the hIAPP_A dataset. BIC is absent from intial investigation using Pomegranate, as this is not calculated automatically as in SciKitLean. BIC calculations in Pomegranate are described subsequently. 

![](./images/Fig7.png)

<strong>Figure 6. 2-dimensional Gaussian mixture modelling of hIAPP_A data using the Pomegranate GeneralMixtureModel module for 2 - 5 components.</strong> Subplot descriptions are as for Figures 3 and 5. Minimum standard deviations for x and y were set to 0.5 and 8, respectively. 
<br>
<br>
Although the model fit was greatly improved, manual evaluation of minumum standard deviations for each dataset was impractical, especilly with a view to software design. The next step was to interpolate the data in both voltage and time dimensions and see whether this allowed successful fitting with little or no adjustment of standard deviation settings. 
##################################################################################################################################
There are a variety of python packages for multivariate data interpolation listed at SciPy, of which nearestNDinterpolator, CloughTocher2DInterpolator interp2, RectBivariateSpline, interpn and RegularGridInterpolator were appropriate for grid-like data. 
Being familiar with the scipy package interp1D, I decided to start invetigations into optimal 2D data interpolation using Interp2D. Interp2D accepts x(arrival time), y(volatge) and z(intensity) data in array-like formats. Data were smoothed to 200 points in both x and y dminsions, which proved to be a reasonable balance between smoothing and computational time. The ensuing density plot (Fig. 7A) was comparabale to the original data (Fig.2A, E) but low intensity, diffuse artefacts were observed at some peripheries of the data grid (Fig. 7A). Neither parameter adjustment, allowing y-values to be interpolated below zero nor clipping grid values at zero could remove these artefacts of interpolation. Therefore the rect bivariate spline interpolation method was attempted next. The data was reshaped into 1D arrays for X and Y data, and one 2D array for intensity values and default parameters were used. 

![](./images/Fig8.png)

<strong>Figure 7. Interpolation of arrival time, voltage and intensity data in the hIAPP_A dataset using Interp2D and RectBivariateSpline interpolation.</strong> In Figure 7A, data from each dimension was formatted as an array and interpolated using Interp2D with the following parameters altered from default: kind = 'cubic', fill value = 0.0. In Figure 7B data was interpolated using the RectBivariateSpline function with parameters were set as: bbox=[None, None, None, None], kx=3, ky=3, s=0. All other parameters ere left as default. For both methods 200 data points were interpolated between 55 and 80 ms and between -10 and 70V in the time and voltage dimensions.
<br>
<br>
######################################################################################################################################
Comparison of the RectBivariateSplie interpolation (Fig. 8B) with the original data (Fig. 3) suggested the interpolation using RBVspline method had been successful (Fig. 9). I therefore continued to fit Gaussian Mixtures to the data with Pomegranate, visualising the original data (Fig. 9A), modelled data (Fig. 9B), means and standard deviations as in previous figures. The model initially appeared to be a good fit as the modelled data was visually similar to the original data (Fig. 9A, B). This fit was tested more robustly by plotting the residuals between the original and recreated data, with positive residuals coloured in red and negative ones in blue (Fig. 9C). 
<br>

![](./images/Fig9.png)

<strong>Figure 9. Interpolation of hIAPP_A data using the RectBivariateSpline method from SciPiy.</strong> Figure 9A shows a density plot of the original data. Fitted means and standard deviations are overlayed on data fitted using Pomegrate (Fig. 9B). The original and modelled data were normalised, subtracted and the residuals plotted, using a hot-cool color map to indicate postive-negative values, along with the RMSD value (Fig. 9C). The optimal number of components was estimated using results from Figure 14. 
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

<strong>Figure 11. Interpolation of hIAPP_B data using the RectBivariateSpline and CloughTocher interpolation methods from SciPy.</strong> RMSD for each method is in black and green text, respectively. Descriptions for Figs. 11A, B and C are as for Figure 9.
<br>
<br>
<br>

![](./images/Fig12.png)

<strong>Figure 12. Interpolation of hIAPP_B data using the CloughTocher interpolation method from SciPy.</strong> RMSD for each method is in black and green text, respectively.  Descriptions for Figs. 12A, B and C are as for Figure 9.
<br>
<br>
<br>

![](./images/Fig13.png)

<strong>Figures 13. Interpolation of hIAPP_B data using the CloughTocher interpolation method from SciPy.</strong> RMSD for each method is in black and green text, respectively.  Descriptions for Figs. 13A, B and C are as for Figure 9.
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
<br>
<br>

In the current implemntation of the code, if the most intense peak is more than 12 y-units away from the first peak on the Y-axis (as is the case in hIAPP_sliceB), this peak is still chosen as the parent peak but a warning is triggered advising the user to examine the results manually. Again, with more test datasets, this trigger-threshold could be much better defined. 

![](./images/Fig17.png)
<strong>Figure 18. Identification of parent and child peaks in the hIAPP_B dataset where identification of the parent peak is not straightforward, so a warning message is triggered.</strong> Fig. 18A, B and C descriptions are as for Fig. 16A, B and C. 
<br>
<br>
Taken together, these results show that Gaussian Mixture Modelling provides a mechanism for deconvoluting the signal from circular ion mobility mass spectrometry, although the large data range means than modules enabling weighting of datapints, such as Pomegranate, are more sutabel than more commonly used modeules such as SciKitLearn. Sucessfull Gaussian modelling is subject to data interpolation inbetween the selected time points and voltages, otherwise the data is not continuous and cannnot be modelled using this method. As evidenced by these results, the interpolation method used can have an appeciable effect on how well Gaussians can be fitted to the data. These results also start to show how data input, processing and results output could be incorporated into software for modelling circular ion mobility data. 

