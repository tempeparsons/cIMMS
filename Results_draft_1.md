# <center><strong>Interpretation of cyclic ion mobility mass spectrometry data using a Gaussian Mixture Model</strong></center>

####  AUTHOR: DR. HARRIET T PARSONS
####  SUPERVISOR: PROF. KONSTANTINOS THALASSINOS, UNIVERSITY COLLEGE LONDON
<br>

#### This report is the result of my own work, unless explicitly indicated.
<br>

<strong>MSC BIOINFORMATICS WITH SYSTEMS BIOLOGY PROJECT REPORT<br>
DEPARTMENT OF BIOLOGICAL SCIENCES,<br>
BIRKBECK COLLEGE,<br>
UNIVERSITY OF LONDON<br></strong>
<br>
<br>
Abstract...............................................................................................................................2
<br>
Acknowledgements........................................................................................................2
<br>
Introduction.......................................................................................................................2
<br>
Methods of investigation and analysis...................................................................4
<br>
&nbsp;&nbsp;&nbsp;&nbsp;Generating datasets..................................................................................................4
<br>
&nbsp;&nbsp;&nbsp;&nbsp;Exploring the data in one dimension.................................................................4
<br>
&nbsp;&nbsp;&nbsp;&nbsp;Aplying a GMM to two dimensional data........................................................4
<br>
&nbsp;&nbsp;&nbsp;&nbsp;Investigating different interpolation techniques and measuring goodness of fit........5<br>
&nbsp;&nbsp;&nbsp;&nbsp;Representing changes in shape and intensity of Gaussians over time and voltage....5<br>
Results..................................................................................................................................5<br>
&nbsp;&nbsp;&nbsp;&nbsp;Intial exploration using one-dimensional data...............................................5<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Figures 1, 2...........................................................................................................5, 6<br>
&nbsp;&nbsp;&nbsp;&nbsp;Clustering using the SciKitLearn GMM module in two dimensions.......7<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Figures 3 - 5.....................................................................................................8 - 10<br>
&nbsp;&nbsp;&nbsp;&nbsp;Clustering in two dimensions using the Pomegranate module.............10<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Figure 6....................................................................................................................11<br>
&nbsp;&nbsp;&nbsp;&nbsp;Investigating different interpolation techniques and measuring goodness of fit.....12<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Figures 7 - 13................................................................................................12 - 15<br>
&nbsp;&nbsp;&nbsp;&nbsp;Representing changes in shape and intensity of Gaussians over time and voltage.16<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Figures 14 - 17...............................................................................................17, 18<br>
Discussion and conclusions........................................................................................19
<br>
References.........................................................................................................................20
<br>
<br>

## <strong>Abbreviations</strong>
IMMS - ion mobility mass spectrometry
<br>
IAPP - islet amyloid polyprotein
<br>
<br>

## <strong>Abstract</strong>
Ion mobility mass spectrometry can yield structural stability information from proteins whose structure is incompatible with other techniques, particularly when applied to comparative studies between sequentially homologous proteins. Structural differences result in comparably different collisional cross sections (CCSs) of proteins during ion mobility, particularly if partial unfolding is induced by an electrical disruption. CCSs are derived from deconvoluting the arrival times of molecules at the MS detector. Deconvolution reveals the Gaussian peaks associated with different structural species present in the sample. Cyclic IMMS is a novel, higher-resolution form of this technique. Ions can travel further and each distinct structural species, and its products, can be selected for repeated analyses. The currently available IMMS software suites are not suited to the large volume of data produced by repeated analyses of unfolding species from both the original and product molecules. This study aims to develop methods for deconvoluting cyclic IMMS arrival time data using freely available Gaussian Mixture Modelling tools in the Python programming language, thereby providing a basis for future software development. Arrival times from two unfolding products of rat and human islet amyloid polypeptide (IAPP) were investigated. Structural aggregation of IAPP is associated with type 2 diabetes but only in species intolerant of excessive food intake such as humans. A trial of different interpolation and modellin modules revealed Clough Tocher interpolation and Pomegranate GMM as the most appropriate. A bespoke script successfully identifed parent and child peaks in three datasets. Results indicated greater structural stability of IAPP in rat, a tolerant species resistant to type 2 diabetes.
<br>
<br>

## <strong>Acknowledgements</strong>
I would like to thank Aisha Ben-Younis for generating and supplying me with data and Dr. Tim Stevens for introducing me to Pomegranate and explaining the Standard Deviation Classes to me. 
<br>
<br>

## <strong>Introduction</strong>

<em>A medical context for ion-mobility structural studies:</em> <br>
Type 2 diabetes is one of the leading causes of disability worldwide. Its incidence has increased rapidly in the last few decades and it continues to do so (Moore et al.). Human islet amyloid polypeptide (hIAPP) is stored in the beta cell along with insulin secretory granules and plays a role in regulating glucose metabolism (Akter et al.). The accumulation of hIAPP aggregates has been shown to correlate with the severity of type 2 diabetes and studies suggest the amyloid fibrils are toxic to beta cells - but that the pre-fibril form where several soluble monomers have formed a soluble oligomer appears even more toxic (Moore et al.). Understanding the soluble structure of hIAPP would provide insights into how these oligomers are formed and why they are cytotoxic. Although structures have been derived for various fibrillar forms (Cao et al.), hIAPP is an intrinsically disordered protein whose monomeric or oligomeric structure cannot be elucidated through crystallography.

<em>Ion-mobility mass spectrometry:</em> <br>
One technique that has proved important in helping elucidate the structures of complete or partially disordered proteins is ion-mobility (Dodds and Baker). In structural IM, a purified protein, complex or other molecule is delivered into the instrument such that non-covalent interactions within the macromolecule(s), and hence structural information, is retained (Eldrid and Thalassinos; Christofi and Barran). In essence, the technique measures the time taken for a molecule to move through a neutral gas. The more unfolded the macromolecule, the larger the area that can collide with gas molecules, so the longer it takes to arrive at the detector. This area is referred to as the collisional cross section (CCS) (Gabelica et al.).

When ion-mobility is coupled to mass spectrometry IM can be performed on a selected mass/charge (m/z) window (Christofi and Barran). This increased resolution of peaks within the arrival time distribution data because, although structural IM is performed on a purified solution, a large array of conformers and charge states are present. The applicability of ion-mobility has been further increased by combining it with other structural evidence such as molecular dynamics or crystallography datasets. Here, the CCS calculated from arrival time data can show which theoretically derived structures are correct or reveal the true mobility range of a crystal structure (Christofi and Barran; Moore et al.). IM-MS can yield very precise structural information in comparison studies where the CCS obtained from comparative studies of e.g. proteins with point mutations or highly similar proteins from different species show directly the sequence responsible for certain unfolding responses (Christofi and Barran). Structural integrity can be interrogated yet further by introducing an external disruption such as a voltage, or ramped sequence of voltages, and measuring the change in population of unfolding species (Allison, Barran, Benesch, et al.; Fernandez‑Lima et al.). 

<em>The importance of cyclic ion-mobility mass spectrometry:</em><br>
These advances have dramatically increased the resolution and applicability of IM-MS but are inherently limited in the sense that the unfolding products of the parent protein must be examined as a whole. It is not possible, for example, to selectively trap multiple unfolding products and investigate their stability individually. This way the structural routes from parent to particular product could be traced, which is critical for understanding formation of amyloid fibrils. The advent of cyclic IM-MS has changed this (Eldrid and Thalassinos). In this technique, the linear drift tube is replaced with a cyclic IM device. Not only does the longer drift distance increase resolution, but the point of ion entry/exit can store ions of a selected mobility i.e. an ion 'slice' from the mobility circuit. Selected, stored ions can be returned to the mobility circuit and unfolding can be induced by a voltage before or after the circuit, providing great precision and flexibility in experimental design and data accumulation (Giles et al.). 


<em>The scope and objectives of this study:</em><br>
In order for arrival time data to be converted into CCS's, the total arrival time spectrum must be deconvoluted into its component peaks, as each peak represents an unfolding structure (Allison, Barran, Benesch, et al.). A large protein or protein complex subject to several voltage steps may contain many peaks, of which each mean and variance will represent a different degree of unfolding. 
<br>
In this study I compare structural stability of two slices of IAPPs from human and from rat by identifying the mean and variance of peaks in arrival time data. Rats, and some other species not susceptible to type 2 diabetes, have IAPP variants which do not form aggregates. Although most of the sequence is conserved between al variants, some short, specific regions are associated with aggregation (Wu and Shea; Cao et al.). This, plus the existence of molecular dynamic simulations for IAPP (Moore et al.), make this subject ideally suited to IM-MS. 
<br>
Arrival time data for each slice was obtained prior to this study by passing rat and human parent polypeptides around the cyclic drift track, selecting the two most prominent unfolding products from each, then subjecting these to a ramped voltage. Typically, peak identification would be carried out using one of the available software suites for IM-MS (Allison, Barran, Benesch, et al.; Allison, Barran, Cianférani, et al.). However, data from cyclic IM-MS is more complex for this software to be of practical use, as one parent protein can generate multiple datasets depending on the number of mobility slices selected and voltage ramps performed. 
<br>
This study therefore aims to find a method for identifying peaks which is straightforward enough that it can be easily incorporated into a future piece of software written specifically for cyclic IM-MS. This study also presents some preliminary aspects of data visualisation for this hypothetical software.
<br>
<br>

## <strong>Methods of investigation and analysis</strong>
<strong>All functions referenced in this section can be found at:</strong> https://github.com/tempeparsons/HTP_masters_thesis/blob/main/HTP_cyclicIMS_Methods.ipynb
<br>
<strong>All data files can be found at:</strong> https://github.com/tempeparsons/HTP_masters_thesis
<br>
<br>
<em>Generating datasets:</em>
<br>
Lyophilised rat and human IAPP samples were prepared and analyzed on a cIM QToF (Waters) according to Eldrid, Ben-Younis et al. by Aisha Ben-Younis and other members of the Thalassinos group. Briefly, human and rat IAPP were passed around the cIM. For each species, two arrival time distribution slices (A and B) were ejected from the cIM, stored, then re-injected for further activation. This yielded a single text file for every voltage at which slices were activated, containing arrival time and intensity readings.
<br>

<em>Exploring the data in one dimension:</em>
<br>
The 'extract_data_for1d' function took all individual volatges files from each of the four datasets and returned a dictionary of arrival_time-intensity values keyed by voltage. These were then plotted as simple line plots ('plot_raw_data' function). intensity value had to be converted into  'histogram-like' data for clustering using the SciKitLearn GMM function. In this state, intensity values were represented by a list of arrival time values, the length of which determined the intensity. Data points for each volatge were coloured by component and overlayed on lineplots. 

<em>Aplying a GMM to two dimensional data:</em>
<br>
Gaussian mixture fitting wasattempted with the SciKitLearn GMM module, then the Pomegrante GMM module. Initially, the individual 'histogram-like' voltage datasets were arranged together in one two-dimensional array per species per slice (see line 'xdata += [arrtime[index]]*n' in 'extract_data_for2d' function). The number of values required to represent the highest intensities was large enough to slow processing, so values were divided by 100. The 'XYArrDiv' return object was passed to the 'skl_gmmfit_plot_histdata' function for GMM fitting with SciKitLearn. A 'fake' version of hIAPP_A was also generated where the dynamic range of intensities was reduced manually whilst maintaining peak shape, as judged by eye, as much as possible. Relevant parameters of SciKitLearn's GaussianMixture function were explored fully in the 'explore_sklgmm_params' function.
<br>
Non-zero data was re-extracted from text files for fitting using Pomegranate GMM. Briefly, all voltages and arrival times were sorted, indexed and stored in a dictionary. An array of zeros was filled with intensity values at the corresponding voltage-arrivaltime coordinates ('j = vidx[v], k = aidx[a], input_density[j,k] = intens' in the 'extract_data_gmmfitPOM_plot' function). Standard deviations were defined by initiating two classes (MinStdNormalDistributionX and Y, in 'extract_data_gmmfitPOM_plot' function). The 'GeneralMixtureModel.from_samples' method was used with parameters set as in 'extract_data_gmmfitPOM_plot' function. This method returns data points as log probabilities, which was unlogged and returned to the original grid shape for plotting ('extract_data_gmmfitPOM_plot' function). 

<em>Investigating different interpolation techniques:</em>
<br>
The impact of interpolating data in both the x and y directions was investigated using three methods available in SciPy: interp2d(scipy.interpolate.interp2d.html), rectbivariatespline(scipy.interpolate.RectBivariateSpline.html) and the CloughTocher interpolator(scipy.interpolate.CloughTocher2DInterpolator.html). The 'compare_interp2d_RBVspine_interpolation' function extracted and formatted data into lists of volatges and arrival times, plus a 2D array (time, V) of intensity values. This allowed comparison of the first two methods using density plots. The return objects from this first function were not investigated further. The second function ('extract_data_for_RB_CT_interpolation' function) extracted data into three lists (voltage, arrival time and intensity), with voltage containing a -10 value, corresponding to an empty set of arrival time values, so data could be accurately interpolated in a Gaussian peak close to y=0. These lists were then passed to either the  RBinterp_PomegranateGMM or  CTinterp_PomegranateGMM functions, which returned a large number of objects packaged into a dictionary, that was passed to the 'plot_fit' function for comparison of density plots. 
<br>

<em>Measuring goodness of fit:</em>
<br>
The dictionary returned by the latter two above functions, RBinterp_PomegranateGMM and CTinterp_PomegranateGMM, contained the return objects 'resid' and 'rmsd'. The residuals in 'resid' were calculated by subtracting the sum-normalised interpolated input data from the sum-normalised fitted output data. Root means square deviation (RMSD) was calulated by taking the square root or the mean of the squared residuals. The residuals were plotted using a hot-cool colour scheme to show increases-decreases in fitted data compared to the original, with RMSD values overlayed. The optimal number of components for GMM fitting was estimated using the Bayesian Information Criterion (BIC). Unlike the SciKitLearn GMM module, Pomegranate does not automatically calculate BIC values. These were calculated manualy using ' bic = float(df * np.log(len(smoothxycoords)) - 2.0 * lp)' in any functions involving Pomegranate GMM. This is elaborated in the results section. The 'CTinterpolation_BICoptimisation' function models interpolated data for up to 10 components, with 5 iterations for each number of components. The ensuing BICs and RMSD values were stored in the returned lists. The 'bic_rmsd_plotter' function plotted means and standard errors from these lists. 

<em>Representing changes in shape and intensity of Gaussians over time and voltage:</em>
<br>
Visualisation methods were explored which showed how the shape of interpolated changed over time and voltage. The Clough-Tocher interpolated data grid of 200 x 200 dimensions was sampled at point 100 along both the x and y axes. The intensity data at each of these points was sampled in steps of 10 in the 'slice_plot' function . Grid coordinates of modelled means were used to establish the position of the most intense peak relative to other along the y-axis. A series of logic steps determined which, if any, peak could be identified at the parent peak (parent_child_peaks function). 
<br>
<br>


## <strong>Results</strong>

In this study I applied Gaussian Mixture Modelling (GMM) to cyclic ion mobility mass spectrometry (cIMMS) data from for four test datasets. Each dataset described the behaviour of a peptide hormone from a particular starting structure, after it was exposed to increasing voltages that further disrupted its folding. Islet amyloid polypeptide peptide hormone from human (hIAPP) or rat (rIAPP) was presenting in two starting structures, _A and _B, making four datasets in total: hIAPP_A, hIAPP_B, rIAPP_A, rIAPP_B. As these starting structures were subjectd to inceasing voltages, they unfolded yet further. The presence of a structural species is marked by a sharp peak in ion intensity at the detector. More unfolded structures take longer to arrive at the detector. However, the parameters of each intensity peak (and therefore the nature of the structure) are not immediately obvious from the entire set of arrival time data for all tested voltages, although these peaks can be assumed to have Gaussian distributions. The results below outline a method by which the means and variance of each peak, and likely number of peaks, can be determined from total arrival time data. Often only a single dataset is shown in a figure but all datasets can be plotted using functions in the githb repository.
<br>
<br>
<em>Intial exploration using one-dimensional data:</em><br>
Clustering methods, including GMM, can be applied to one, two or higher dimensional data. The current data comprises meaurements in time, voltage and intensity. If, however, intensity is represented by the frequency of data points at a given time measurement, the data effectively takes on the properties of a histogram and the dimensionality is reduced. If the clustering is performed separately for each voltage, Gaussian peaks can be fitted in a single dimension. Being unfamiliar with GMM, I began investigations into its applicability to this data by performing 1D clustering on each voltage using the GMM module in the SciKitLearn library (Figure 1).  
<br>

![](./images/Fig1.png)

<font size="1"><strong>Figure 1. 1-dimensional Gaussian mixture modelling of hIAPP_A data.</strong> Data from each voltage was reduced in dimensionality by expressing intensity at the number of points at an arrival time, then subjected to 1D clustering using the sklearn.mixture.GaussianMixture module. The following parameters were applied to all voltages: n_components=3, max_iter=100, init_params='kmeans', covariance_type='full', random_state=0. Default settings were used for all other paramters and data points were coloured by whichever component the original data point was fitted to. The relationship between fitted peaks and the total arrival time spectra is shown by superimposing line plots of arrival time and intensity for each voltage. Note the high intensity range on the y-axes of subplots. Note also that colors do not necessarily match a particular component. </font>
<br>
<br>
In the above clustering Gaussian fittings, the number of components was estimated by k-means and was set to an initial value of 3 for all voltages. For each voltage in Figure one, the components to which the 1-dimensional data points at the bottom have been assigned look plausibel given the line plots showing intensity. However, after 20 V the base positions of the components along the x-axis becomes inconsistent. Obviously, the apex of some components will be first detected in a different voltage from the dataset containing their apex. In order to find the real shape of a component, it must be considered in the context of all volatges. Figure 1 therefore shows that these datasets are not appropriate for fitting in one dimension.

The principal of 2D Gaussian fitting is similar to 1D fitting, except that covariance i.e. how much a change along one axis affects change along the other axis, must be carefully considered. The longer a protein is subject to a voltage the more it will unfold. However, for any one data point, and therefore any one Gaussian peak, the variation in voltage is independent of the variation in arrival time.  

Before applying 2D Gaussian fitting, the overall shape of all test datasets was compared to check the likelihood that the same Methods could be applied to all datasets. Intensity was better visualised using a 2D histogram (Fig. 2A, B, D, E) and a contour plot (Fig. 2F, G, H, I), with intensities represented by a heatmap. Overall, there did not seem to be any differences in dataset shape so large as to consider using different fitting methods, although rat data (Fig. 2D, E) looked slightly less gaussian than human data (Fig.2A, B). Importantly, x and y were not the same length in all datasets. Consequently, the modelling code was tested on the same lengthed datasets (hIAPP_A and _B) before the final methods were written into functions which could account for this. 

![](./images/Fig2AtoD.png)

![](./images/Fig2EtoH.png)

<font size="1"><strong>Figure 2. Visualising the shapes of a dataset using 2d histograms and contour maps.</strong> 2D arrays were generated with one layer containing non-zero arrival times proportional to the intensity and another with corresponding voltages. For each dataset this was plotted as a 2D histogram using the numpy.histogram2d function and coloured from highest intensity (yellow) to lowest (black) as follows: hIAPP_A; 2A, hIAPP_B; 2B, rIAPP_A; 2C, rIAPP_B; 2D. Equivalent arrays were generated for plotting contour maps (Figs. 2E - H). </font>
<br>
<br>

<em>Clustering using the SciKitLearn GMM module in two dimensions:</em><br>
A number of parameters in the SciKitLearn GMM module must be optimised for fitting 2-dimensional data. GMM uses the EM algorithm to find the best model for the data. Essentially, the EM algorithm guesses starting means and variances for a dataset where the means, variances and the number of components (or peaks) are unknown. The algorithm creates a new dataset with the guessed means and variances, then calculates the probability that the original data came from Gaussian peaks described by the guessed parameters. The contribution of each data point to this total probability score is weighted by each datapoint's distance from the guessed means. This allows the data fit to be improved; if the guessed means were distant from most points, the summed probabiltiy score would be low. The algorithm then alters the guessed parameters so that the assessed probability of all data points is increased. Once the log liklihood of this summed score is not increasing appreciably between guesses (using a pre-determined threshold), then the best data model is said to have been achieved. Clearly, therefore, the method used for taking the initial guess will profoundly impact the final model. 

SciKitLearn offers three main types of ititialisation method: k-means, a random selection from the original data points or a random selection of points close the mean of the total dataset. Once the method has been chosen, the number of components must be chosen. Ideally this would be the same as the real number of Gaussian peaks but this is, of course, unknown. 

The optimal number of components can be deduced using Bayesian Information Criterion (BIC), for which there is an inbuilt method in SKL. BIC describes how likely a model is given the data, compared to the other models. If the variance in the error between the data and model increases, or if the number of components increases, so too does BIC. The latter point means a large number of initial components is penalised, which can help avoid overfitting unless the model is genuinely complex. If BIC is calculated for a range of components, a steep drop in BIC often indicates the optimal number. 

Covariance type is another parameter to consider. Covariance describes whether changes in one variable e.g. voltage, impact on another variable e.g arrival time. As mentioned, these are expected to be independent. Of the four covariance types provided by SciKitLearn ('full', 'tied', 'spherical' and 'diagonal') 'diagonal' describes independent covariance of x and y - although all convariance types were eventually examined. 
 
Data was arranged as a 2D numpy array, as descried for Figure 2, and 2D GMM was applied. BIC was ascertained for 2 - 5 components. Otherwise, model fitting was evaluated visually, using coloured datapoint plots similar to to Figure 1, density and contour plots. 

![](./images/Fig3.png)

<font size="1"><strong>Figure 3. 2-dimensional Gaussian mixture modelling of hIAPP_A data using the SciKitLearn GMM module.</strong> Between 2 and 5 components were tested, initialised using k-means, with covariance type set to 'diag'. Default settings were used for all other paramters. In Fig. 3A Gaussian cluster are not coloured so that the means and standard deviations are easily visualised. Datapoints from the original data are coloured according their assigned Gaussian component(3B). Fitted data distribution is visualised using density plots (3C) and contour maps (3D). Modelled means are shown in red (3C, D) and the BIC associated with each components number is given in the plot titles (3A). </font>
<br>
<br>
A number of important interpretations were made from this data which directed the subsequent directions of analysis. Firstly, despite the intensity values being divided by 100, this approach was impractically slow owing to the large number of points required to reprepsent the highest intensity readings. 

Secondly, modelling in 2D did not produce even moderately comparable clusters (Fig. 3E-H) to 1D data models (Fig. 1). This was most apparent when using > 3 components (Fig. 3F-H), where components became fitted to voltage datasets rather than the Gaussians spanning them. Consequently the overall data shape (Fig. 3I-P) only ressembled the original data when 2 components (Fig. 3I, M) were used. For completeness, different covariance and initialisation parameters were investigated but none resulted in improved model fits (Fig. 4).
<br>

![](./images/Fig4.png)

<font size="1"><strong>Figure 4. Impact of parameter adjustment on fitting Gaussian peaks to hIAPP_A data, using the SciKitLearn GMM module.</strong> Gaussian mixtures were modelled using data prepared as for Fig.3 but, using a 3 starting components and a set random_state, all possible initialisation methods and covariance types were investigated. Only results for covariance type 'full' and 'diag' are shown here. Parameters are detailed in subplot titles. Clusters are coloured using matplotlib colormap 'viridis' as in Figure 1. </font>
<br>
<br>

The error bars in Figure 3A-D revealed an extremely unequal variance in the x and y directions. If more unequal that assumed by the 'diagonal' covariance type, this could have explained the failure to fit the data. Although time and voltage are continuous variables by nature, in these datasets they were both effectively being used as discreet data, with 200 categories in the time dimension and up to 8 in the voltage dimension. GMM is only suitable for continuous data, so interpoltion of intervening data points in both dimensions was necessary. The very large range in intensitiy measurements (Fig. 1) may also have been compromising the detection of smaller intensity peaks in Figure 3, although this was likely of secondary importance. 
<br>
<br>
<em>Investigating the impact of dynamic range on model fit failure:</em><br>
The contribution of intensity range to the model failure was investigated first, as this could be assessed relatively quickly by artificially lowering the data range. Global adjustments such as logging or sqaure rooting would have removed the Gaussian shape of the data. Instead, artifical data was generated by manually altering voltage sets with the lowest and highest intensity measurements whilst judging maintenance of the peak shape by eye. For ease of comparison, the same analysis was performed as in Figure 3. At 2, 3 and 4 components, the the best fitting model using the artificial data was closer to the original shape of the real data than any model fitted to the real data (Fig. 5). This suggested that the very large data range was compromising the ability of the SciKitLearn GMM function to fit Gaussians to this data. However at 4, and especially 5 components, the same effect was observed as in Figure 3 where the function started to define cluster by voltage rather than arrival times of unfolding proteins; the very large data range seemed to have detectable but marginal impact on model fitting failure. 
<br>

![](./images/Fig5.png)

<font size="1"><strong>Figure 5. 2-dimensional Gaussian mixture modelling of semi-artifical hIAPP_A data using the SKL GMM module.</strong> Source data differed from that shown in Fig. 3 in that files containing the most intense peaks (10 Volts) least intense peaks (60 and 70 Volts) had been artificially lowered and raised, respectively. Otherwise all other parameters and analyses were set as in Figure 3. </font>
<br>
<br>

<em>Clustering in two dimensions using the Pomegranate module:</em><br>
Of immediate importance, however, was the first problem that the number of datapoints representing the greatest intensities meant datasets were enormous and investigating multiple components was becoming unfeasibley slow. The need to represent intensity this way was eliminted by using a different modelling library. Pomegranate is a python package that offers a wide range of probabalistic modelling (pomegranate.readthedocs.io/en/latest/). It also allows data points to be weighted, making it easier to deal with high intensity measurements. Required data inputs are a grid of coordinates for each datapoint in one array and another array containing the weighting of each point. The MultivariateGaussianDistribution function from Pomegranate's GeneralMixtureModel module uses 'full' convariance by default. This is converted to independent x and y, by defining the minimum standard x and y deviation as classes and using these in place of MultivariateGaussianDistribution. The minimum standard deviation in these classes can then be changed - thereby allowing the hypothesis to be tested that very unequal standard deviations in Figure 3 were the principal factor behind model failure.

Using the same visualisation scheme and component numbers as in Figures 3 and 5, Gaussians were fitted to the hIAPP_A dataset using Pomegranate, with datapoints weighted by intensity. Firstly, I attempted to reproduce the results from SciKitLearn in which the fit had failed by using Pomegranate's MultivariateGaussianDistribution function. This indeed did produce a similarly failing fit, indicating the two methods were comparable. Setting minimum standard deviation classes dramatically improved the model (Fig. 6); Gaussians now followed intensities rather than voltage at >2 components and the underlying shape of the data was not changing as components were added. Differences in standard deviations were initially set to the differece in data density between Voltage and Arrival Time. After a few attempts, setting minstdX = 0.5 and minstddevY = 8 was found to yield optimal results for the hIAPP_A dataset. BIC is absent from intial investigation using Pomegranate, as this is not calculated automatically as in SciKitLean. BIC calculations in Pomegranate are described toward the end of the Results section. 

![](./images/Fig6.png)

<font size="1"><strong>Figure 6. 2-dimensional Gaussian mixture modelling of hIAPP_A data using the Pomegranate GeneralMixtureModel module for 2 - 5 components.</strong> Subplot descriptions are as for Figures 3 and 5. Minimum standard deviations for x and y were set to 0.5 and 8, respectively. </font>
<br>
<br>
Although the model fit was greatly improved, manual evaluation of minumum standard deviations for each dataset was impractical, especilly considering software design. The next step was to interpolate the data in both voltage and time dimensions and see whether this allowed successful fitting with little or no adjustment of standard deviation settings. 
<br>
<br>
<em>Investigating different interpolation techniques and measuring goodness of fit</em>
There are a variety of python packages for multivariate data interpolation listed at SciPy, of which nearestNDinterpolator, CloughTocher2DInterpolator interp2, RectBivariateSpline, interpn and RegularGridInterpolator were appropriate for grid-like data. Being familiar with the scipy package interp1D, I decided to start invetigations into optimal 2D data interpolation using Interp2D. Interp2D accepts x(arrival time), y(volatge) and z(intensity) data in array-like formats. Data were smoothed to 200 points in both x and y dminsions, which proved to be a reasonable balance between smoothing and computational time. The ensuing density plot (Fig. 7A) was comparabale to the original data (Fig.2A, E) but low intensity, diffuse artefacts were observed at some peripheries of the data grid (Fig. 7A). Neither parameter adjustment, allowing y-values to be interpolated below zero nor clipping grid values at zero could remove these artefacts of interpolation. Therefore the rect bivariate spline interpolation method was attempted next. The data was reshaped into 1D arrays for X and Y data, and one 2D array for intensity values and default parameters were used. 

![](./images/Fig7.png)

<font size="1"><strong>Figure 7. Interpolation of arrival time, voltage and intensity data in the hIAPP_A dataset using Interp2D and RectBivariateSpline interpolation.</strong> In Figure 7A, data from each dimension was formatted as an array and interpolated using Interp2D with the following parameters altered from default: kind = 'cubic', fill value = 0.0. In Figure 7B data was interpolated using the RectBivariateSpline function with parameters were set as: bbox=[None, None, None, None], kx=3, ky=3, s=0. All other parameters ere left as default. For both methods 200 data points were interpolated between 55 and 80 ms and between -10 and 70V in the time and voltage dimensions.</font>
<br>
<br>
Comparison of the RectBivariateSplie interpolation (Fig. 7B) with the original data (Fig. 3I, M) suggested the interpolation using RBVspline method had been successful. I therefore continued to fit Gaussian Mixtures to the data with Pomegranate, visualising the original data (Fig. 8A), modelled data (Fig. 8B), means and standard deviations as in previous figures. This fit was tested more robustly by plotting the residuals between the original and recreated data, with positive residuals coloured in red and negative ones in blue (Fig. 8C). 

![](./images/Fig8.png)

<font size="1"><strong>Figure 8. Interpolation of hIAPP_A data using the RectBivariateSpline method from SciPiy.</strong> Figure 8A shows a density plot of the original data. Fitted means and standard deviations are overlayed on data fitted using Pomegrate (Fig. 8B). The original and modelled data were normalised, subtracted and the residuals plotted, using a hot-cool color map to indicate postive-negative values, along with the RMSD value (Fig. 8C). The optimal number of components was estimated using results from Figure 13. </font>
<br>
<br>
The residual plot indicated a reasonable fit between the modelled and original data although clear disparities in peak shape remained (Fig. 8C). I therefore decided to try one more interpolation method available from SciPy, the CloughTocher2DInterpolator, to see if the fit could be improved any further.  The CloughTocher interpolated accepts a 2D array of x,y data and separate 1D array of z data. The data was reshaped accordingly, and default parameters were used. Residuals in Figure 9C showed a small but noticeable improvement compared to Figure 10C (RMSD 0.0000202 vs x0.0000188). Given that the improvement in fit was not huge, I tested the performance of each interpolation method on the three remaining datasets, hIAPP_sliceB (Fig. 10), rIAPP_sliceA (Fig. 11) and rIAPP_sliceB (Fig. 12). 
 

![](./images/Fig9.png)

<font size="1"><strong>Figure 9. Interpolation of hIAPP_A data using the CloughTocher interpolation method from SciPy.</strong> Figures 9A, B and C are as for Figure 8, excepting the interpolation method. </font>
<br>
<br>

![](./images/Fig10.png)

<font size="1"><strong>Figure 10. Interpolation of hIAPP_B data using the RectBivariateSpline and CloughTocher interpolation methods from SciPy.</strong> RMSD for each method is in black and green text, respectively. Descriptions for Figs. 11A, B and C are as for Figure 9.</font>
<br>
<br>

![](./images/Fig11.png)

<font size="1"><strong>Figure 11. Interpolation of rIAPP_A data using the RectBivariateSpline and CloughTocher interpolation method from SciPy.</strong> RMSD for each method is in black and green text, respectively.  Descriptions for Figs. 12A, B and C are as for Figure 9.</font>
<br>
<br>

![](./images/Fig12.png)

<font size="1"><strong>Figures 12. Interpolation of rIAPP_B data using thee RectBivariateSpline and CloughTocher interpolation method from SciPy.</strong> RMSD for each method is in black and green text, respectively.  Descriptions for Figs. 12A, B and C are as for Figure 9.</font>
<br>
<br>

The CloughTocher interpolater consistently performed better than the RectBivariate Spline interpolation method, judging by RMSD of original vs. modelled data (Figs. 9 - 12); this therefore became the method of choice. Having established a plausible method which could run reasonably quickly, the next challenge was to consider how it could be implemented in some hypothetical software, and which graphical outputs would be most useful. A pleasing outcome of the interpolation was that Gaussians could be fitted without having to specify a minimum standard deviation for any of the datasets tested, so standard deviation no longer needed to be considered as a parameter. For the majority of the Pomegranate modelling, however, I had been visually inspecting graphical output to estimate the optimal number of components and adjusting this on a trial and error basis, which is not a possible approach for automated software. 

I therefore implemented a method for automating measurement of the BIC and RMSD between original and modelled data for each dataset, on which software users could base their choice of component number (assuming a limit of 10 components). As mentioned, Pomegranate does not have an inbuilt method for calculating BIC. BIC has been introduced briefly already but must be now be expained in more detailed. BIC is defined as: BIC = k ln(n) - 2 ln(L) where L is "the maximized value of the likelihood function of the model" (https://en.wikipedia.org/wiki/Bayesian_information_criterion), or the probability of the data having been generated by the best fitting model, k is the number of components and n is the number of data points being fitted in the model. In log space the goodness of model fit is therefore directly proportional to the number of components and size of the dataset. As L increases, so the value of BIC decreases unless k is large - hence how the BIC value is penalised by a large number of components (see Methods: bic = float(df * np.log(len(smoothxycoords)) - 2.0 * lp). The relative decrease in BIC was emphasised by plotting the change in gradient between n components, rather than the BIC itself (Fig. 13A). 

![](./images/Fig14.png)

<font size="1"><strong>Figure 13. Assessment of optimal number of components for each of the four test datasets</strong> BIC (13A) and RMSD (13B) between original and modelled data was calculated for 2 - 10 components for each of the test datasets. Figure 13 shows results for hIAPP_A. Other datasets can be plotted using functions in the Methods section. For all plots, error bars show standard error of the mean for 5 repeats.</font>
<br>
<br>
Results showed general, if not completely consistent, agreement between BIC (Fig. 13A) and RMSD (Fig. 13B). A scenario in which choice of components was fully automated might involve, for example, calculating the number of compontents at which the previous gradient was less than half the current one. However, given the sometimes inconsistent results, the safest policy may be to present the user with all the data in Figure 13 for them to decide the optimal number of components.
<br>
<br>

<em>Representing changes in shape and intensity of Gaussians over time and voltage:</em><br>
Having developed a basic method for fitting Gaussians and asessing the quality of fit, I next considered how else this data could be usefully visualised. In addition to the plots already presented, I investigated how to present horizontal and longitudinal sections through the interpolated data at user-select time points, plus an assessment of overall protein stability throughout the experiment. 

Intensity profiles were visualised over voltage and time by 'slicing' data across Y (Fig. 14A) and X (Fig. 14B) grid axes. Slice size and position was easily defined by selecting rows or columns in the 2D numpy array containing the data, in a way that could eventually be controlled through an interactive interface. 

![](./images/Fig15.png)

<font size="1"><strong>Figure 14. Slices horizontally (Fig. 14A) or vertically (Fig. 14B) across the data grid of the CloughTocher-interpolated hIAPP_A dataset prior to Gaussian fitting.</strong> Fig. 14A shows horizontal cross sections from the start to halfway up the Y-axis of the density plot shown in Fig. 9. Each line represents intensities at a cross section in increments of 10. Fig. 14B shows vertical cross sections of the data grid along the X-axis. Data was sectioned over columns equivalent to 55 to 67 mins in Fig. 10, in increments of 5.</font>
<br>
<br>  

Finlly, I decided to present an overall view of protein stability by comparing the proportion of the initial protein peak to all unfolding/unfolded peaks as the voltage was increased. This required that an initial parent peak, present at 0V, was correctly defined and that the area under this peak was measured separately from all the child peaks. Although conceptually straightforward, correct identification of the parent peak required some thought. An obvious first step would be to define the parent peak as the first along the Y(Volts)-axis and X(time)-axis. This approach worked in datasets where the first peak was the most intense peak, such as rIAPP_sliceA and hIAPP_sliceA. However, in rIAPP_sliceB low-intensity peaks were detected at a lower voltage and/or time than the parent peak. 

![](./images/Fig16.png)
![](./images/Fig18.png)

<font size="1"><strong>Figure 15. Density and line plots showing the intensities of the parent peak compared to child peaks.</strong>. This figure shows plots for hIAPP_A and rIAPP_A, in which identification fo the parent peak was straightforward as it was both the first and the most intense peak in the y-direction. All Gaussians identified in the fitted data and their means are shown in 15A, E. The child (15B, F) and parent (15C, G) peaks are visualised separately. In the absence of the parent peak, the color map is re-scaled to the intensity of the child peaks (15B, F). Finally, the relative intensities of parent and child peaks are shown (15D, H). </font>
<br>
<br>
The more complex the starting sample, the more plausible such a scenario becomes. Therefore the code was adapted to check whether the first peak along the Y-axis was the most intense by comparing positions in np.argsort lists of mean intensities and mean y-coordinates. When this was not the case, the code then asked whether the y-coordinates for the highest intensity mean was less than 12 y-unit away from the first peak. If so, this more intense peak is accepted as the parent peak. The threshold of 12 y-units is based on the voltage required to produce a substantial second peak in these four test datasets. It is difficult to define a more sophisticated threshold based on only four test datasets, although one can be envisaged which uses both X and Y coordinates based on a radius around the parent peak from e.g. 20 datasets. 

![](./images/Fig19.png)
<font size="1"><strong>Figure 16. Identification of parent and child peaks in the rIAPP_B dataset where a less intense peak, close to the genuine parent peak, occurs first in the y-direction.</strong> Fig. 16A, B and C ddescriptions are as for Fig. 15A, B and C. </font>
<br>
<br>
In the current version of the code, if the most intense peak is more than 12 y-units away from the first peak on the Y-axis, as was the case in hIAPP_sliceB, this peak is still chosen as the parent peak but a warning is triggered advising the user to examine the results manually. Again, with more test datasets, this trigger-threshold could be much better defined. 

![](./images/Fig17.png)
<font size="1"><strong>Figure 17. Identification of parent and child peaks in the hIAPP_B dataset where identification of the parent peak is not straightforward, so a warning message is triggered.</strong> Fig. 17A, B and C descriptions are as for Fig. 15A, B and C. </font>
<br>
<br>
Taken together, these results show that Gaussian Mixture Modelling provides a mechanism for deconvoluting the signal from circular ion mobility mass spectrometry. The large data range means than modules enabling weighting of datapints, such as Pomegranate, are more suitable than more commonly used modeules such as SciKitLearn. Sucessfull Gaussian modelling is subject to data interpolation inbetween the selected time points and voltages, otherwise the data is not continuous and cannnot be modelled using this method. As evidenced by these results, the interpolation method used can have an appeciable effect on how well Gaussians can be fitted to the data. These results also start to show how data input, processing and results output could be incorporated into software for modelling circular ion mobility data. 
<br>
<br>

## <strong>Discussion</strong>

This study investigates suitable methods for fitting Gaussian peaks to arrival time data from circular ion mobility mass spectrometry. Being able to quickly identify peaks in intensity data collected over time and at different voltages, and describe changes in the properties, allows researchers to measure changes in protein conformation and, therefore, stability. Contemporary IM-MS software suites (Migas et al. 2017; Lee et al. 2021; Allison et al. 2020) are not designed for the large numbers of datasets that cyclic IM-MS can generate. By designing methods that are fast and based on readily available python packages with a flexible set up, the functions presented in these methods can be easily modified, combined or scaled to any number or shape of data in an ‘arrival time - intensity – voltage’ format. Several graphical output styles and functions for measuring goodness of fit using residuals, BIC and RMSD are presented here (Figs. 6 – 13) that could be incorporated into some future cyclic IM-MS software.
 <br>

This study has achieved, using a general mixture model and Gaussian distributions, a good fit between the intensity data and data that has been modelled around the means of detected intensity peaks.  However, residual plots (Figs. 8 – 12),  suggest that fits could be improved. Interpolating using the Clough Tocher method (Clough and Tocher 1965) yielded a moderate, but consistent, improvement in RMSD compared to the RectBivariateSpline method. Interpolation techniques have not been exhaustively compared in this study. It is worth noting the 'wobble' in intensity slices in Figure 14A. CloughTocher interpolation is good at interpolating local trends in data (Wang 2004), so may possibly have amplified small, local variance. Techniques for both unstructured and on-grid data are appropriate in this case, meaning that the scipy.interpolate library contains at least five other suitable methods with the same data shape requirements as CloughTocher2DInterpolator (https://docs.scipy.org/doc/scipy/reference/generated/scipy.interpolate.CloughTocher2DInterpolator.html): griddata (https://docs.scipy.org/doc/scipy/reference/generated/scipy.interpolate.griddata.html), linearNDinterpolator (https://docs.scipy.org/doc/scipy/reference/generated/scipy.interpolate.LinearNDInterpolator.html), interpn (https://docs.scipy.org/doc/scipy/reference/generated/scipy.interpolate.interpn.html) and  regulargridinterpolator (https://docs.scipy.org/doc/scipy/reference/generated/scipy.interpolate.RegularGridInterpolator.html). Working these methods into the existing interpolation function could be a worthwhile future investigation. Another possibility for improving fit could be to fit Gaussian-like distributions, such as Poisson or Gamma, both of which are possible using the Pomegranate general mixture model (https://pomegranate.readthedocs.io/en/latest/). It is also possible to use other methods for fitting peaks in Pomegranate such as Markov Chain MonteCarlo, although this is far beyond the scope of the present study. 
<br>

Although this analysis has been primarily concerned with methods, rather than calculating CCS or attributing changes in CCS to structural features preliminary conclusions about the structural stability of IAPP are nonetheless possible. 
<br>

After subjecting both human and rat IAPP (hIAPP, rIAPP) to multi-pass ion mobility separations, two distinct structural species (A and B) were sampled from each. The rat version appeared more stable than the human one. rIAPP_A showed consistent arrival time of ~ 62 ms for all voltages (Figs. 11) compared to hIAPP_A (Fig. 9) where arrival tme change from 61 to 64 ms as the voltage increased above 20 V. The absence of shift is rats is consistent with the notion of rat IAPP stability and that rats, along with other species such as pig ,can tolerate excessive food intake (Wu and Shea 2013). 
Compared to hIAPP_B (Fig. 10), it is interesting to note that both hIAPP_A and rIAPP_A show a decrease in total signal at higher voltages, even if the arrival time of detectable rIAPP_A did not change. This suggests generation of unfolding products below the limit of detection, perhaps because CID, as well as CIU, occurred at the highest voltages.
<br>

The hIAPP_B unfolding product of the original hIAPP appears considerably more stable than hIAPP_A, as arrival times do not increase with voltage. The initial arrival time for hIAPP_B is slightly later than hIAPP_A (~63 ms vs. ~61 ms), which could indicate that whatever unfolding events could occur has already occurred and hIAPP_B had a greater CCS than hIAPP_A. This would be consistent both the the absence of shift in arrival time, and the continuation of total signal intensity at the lowest and highest voltage for hIAPP_B.
<br>

Other aspects of hIAPP_B are less straightforward to interpret. Whilst total signal intensity was sustained at the lowest and highest voltages in hIAPP_B, signal intensity decreases at 40V and all but disappears between ~10 and ~20 V. This is visualized most clearly in Figure 17, where the near-absence of difference in arrival times between peaks in signal intensity has resulted in a warning being triggered when the ‘parent_child_peaks’ function, which measures the parent:child peak ratio, is run. It is difficult to envisage a scenario where a single child peak can be larger than its parent. Conceivably, a parent polypeptide’s stability could have a completely non-linear relationship with voltage, such that at 30 and 50 V either the parental structure was unaffected or all children coincidentally had the same arrival time. At 20 and 40 V structures became completely disassociated and were not detected.
<br>

rIAPP_B shows a similar response to rIAPP_A (Figs. 12, 16), albeit with a later initial arrival time, though this is not immediately obvious, as several low-intensity child peaks produced at high voltages have shorter arrival times than their parent. This is perhaps most easily explained by child structures having completely and partially collapsed, as their CCS is lower than their parent’s. rIAPP_B therefore seems to be less stable than rIAPP_A. The unfolding products must be different from those of hIAPP_A, as no collapse was apparent in Fig. 9. 
<br>

In summary, results in this study show how the computational challenge of large cyclic IMMS datasets can be approached, although without detailed examination of sequences and CCS's, and using only a handful of test datasets, it is not possible to extend this analysis to any biological conclusions.
<br>
<br>


## <strong>References</strong>
Akter, Rehana, et al. “Islet Amyloid Polypeptide: Structure, Function, and Pathophysiology.” Journal of Diabetes Research, vol. 2016, 2016, p. 2798269, doi:10.1155/2016/2798269.
<br>

Allison, Timothy M., Perdita Barran, Sarah Cianférani, et al. “Computational Strategies and Challenges for Using Native Ion Mobility Mass Spectrometry in Biophysics and Structural Biology.” Analytical Chemistry, vol. 92, no. 16, Aug. 2020, pp. 10872–80, doi:10.1021/acs.analchem.9b05791.
<br>

Allison, Timothy M., Perdita Barran, Justin L. P. Benesch, et al. “Software Requirements for the Analysis and Interpretation of Native Ion Mobility Mass Spectrometry Data.” Analytical Chemistry, vol. 92, no. 16, Aug. 2020, pp. 10881–90, doi:10.1021/acs.analchem.9b05792.
<br>

Cao, Qin, et al. “Cryo-EM Structure and Inhibitor Design of Human IAPP (Amylin) Fibrils.” Nature Structural & Molecular Biology, vol. 27, no. 7, July 2020, pp. 653–59, doi:10.1038/s41594-020-0435-3.
<br>

Clough, R.W. and Tocher, J.L. (1965) Finite Element Stiffness Matrices for Analysis of Plates in Bending. Proceedings of Conference on Matrix Methods in Structural Analysis, 1, 515-545.
<br>

Christofi, Emilia, and Perdita Barran. “Ion Mobility Mass Spectrometry (IM-MS) for Structural Biology: Insights Gained by Measuring Mass, Charge, and Collision Cross Section.” Chemical Reviews, vol. 123, no. 6, Mar. 2023, pp. 2902–49, doi:10.1021/acs.chemrev.2c00600.
<br>

Dodds, James N., and Erin S. Baker. “Ion Mobility Spectrometry: Fundamental Concepts, Instrumentation, Applications, and the Road Ahead.” Journal of the American Society for Mass Spectrometry, vol. 30, no. 11, Nov. 2019, pp. 2185–95, doi:10.1007/s13361-019-02288-2.
<br>

Eldrid, Charles, and Konstantinos Thalassinos. “Developments in Tandem Ion Mobility Mass Spectrometry.” Biochemical Society Transactions, vol. 48, no. 6, Dec. 2020, pp. 2457–66, doi:10.1042/BST20190788.
<br>

Fernandez-Lima, F. A., et al. “Note: Integration of Trapped Ion Mobility Spectrometry with Mass Spectrometry.” The Review of Scientific Instruments, vol. 82, no. 12, Dec. 2011, p. 126106, doi:10.1063/1.3665933.
<br>

Gabelica, Valérie, et al. “Recommendations for Reporting Ion Mobility Mass Spectrometry Measurements.” Mass Spectrometry Reviews, vol. 38, no. 3, May 2019, pp. 291–320, doi:10.1002/mas.21585.
<br>

Giles, Kevin, et al. “A Cyclic Ion Mobility-Mass Spectrometry System.” Analytical Chemistry, vol. 91, no. 13, July 2019, pp. 8564–73, doi:10.1021/acs.analchem.9b01838.
<br>

Moore, Sandra J., et al. “Characterisation of the Structure and Oligomerisation of Islet Amyloid Polypeptides (IAPP): A Review of Molecular Dynamics Simulation Studies.” Molecules (Basel, Switzerland), vol. 23, no. 9, Aug. 2018, doi:10.3390/molecules23092142.
<br>

Wu, Chun, and Joan-Emma Shea. “Structural Similarities and Differences between Amyloidogenic and Non-Amyloidogenic Islet Amyloid Polypeptide (IAPP) Sequences and Implications for the Dual Physiological and Pathological Activities of These Peptides.” PLoS Computational Biology, vol. 9, no. 8, Aug. 2013, p. e1003211, doi:10.1371/journal.pcbi.1003211.
