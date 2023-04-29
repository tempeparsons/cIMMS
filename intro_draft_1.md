Introduction. 

Type 2 diabetes is one of the leading causes of disability worldwide. Its incidence has increased rapidly in the last few decades and it continues to do so. Human islet amyloid polypeptide (hIAPP) is stored in the beta cell along with insulin secetory granules, and plays a role in regulating glucose metabolism. The accumulation of hIAPP aggregates has been shown to correlate with the severity of type 2 diabetes and studies suggest the amyloid fibrils are toxic to beta cells - but that the pre-fibril form where several soluble monomers have formed a soluble oligomer appears even more toxic. Understanding the soluble structure of hIAPP would provide insights into how these oligomers are formed and why they are cytotoxic. Although structures hav been derived for various fibrillar forms, hIAPP is an intrinsically disordered protein whose monomeric or oligomeric structure cannot be elucidated through crystallography.

One technique that has proved important in helping elucidate the structures of complete or partially disordered proteins is ion-mobility, coupled to mass spectrometry. IM is fundamentally different from MS or tandem MS in that rather than trying to identify proteins in a complex mixture by detecting the peptides, or peptide fragments, a purified protein, complex or other molecule is delivered into the instruemnt. The non-covlent interactions within the macromolcule(s) are retained so higher-level structural information is obtained. In essence, the technique measures the time taken for a molecule to move through a gas-filled chamber. The gas is neutral, so mobility depends entirely on the number of collisons with gas molecules. The more unfolded the macromolecule, the larger the area that can collide with gas molecules, so the longer it takes for the macromolecule to arrive at the detector. The collisional cross section (CCS) is a measurement of the area that collides with the gas. When combined with other sources of information, as descibed later, CCS informs on the different structures a protein or complex can assume. 

Coupling ion-mobility to mass spectrometry meant that, in the case of structural studies on purified protein(s), IM can be performed on a select mass/charge (m/z) window. This increases resolution, as a number of conformers and charge states will be present within a purified protein solution as it is delivered into the instrument.

explain how combinign im ms with other stuff e.g. MD has yielded very useful strcutrla info . then go onto introduce the concept of cyclic im ms and how this is effectively tandem. 






What is ion mobility MS
Why is it useful
Brief history culminating in cyclic ion mobility mass spec.

CIMMS can be particularly useful for biological applications where understanding the structural differences between stability of different but related protein species.
Explain the particular protein in this case. 
Explain why couldn't do this with e.g. crystallography.
Explain how the data generated from cIMMS contributes to structural understanding such that we can work out why similar protein structures have different stability. 

What software is out there for IMMS and why not that much.
What are its limitations when it comes to cyclic IMMS?

What is the exact nature the of problem that needs to be resolved for cyclic IMMS?
Dimensionality of data
Slices and different voltages leading to proliferating data.

Therefore ultimately need to write specific software for cIMMS.
This software needs to be able to accept as input intensity vs time plots for a series of volatages per slice of protein species.
Then it needs to find the minimum number of protein unfolding products that can explain the changes in intensity recorded over time and voltage. 

It is reasonable to assume that the presence (intensity) of the parent protein species and unfolding products will follow a Gaussian distribution. 
Explain why.
Therefore, the peaks of parent and unfolding product can be modelled using a machine learning distribution technique such as Gaussian Mixture Modelling. 

A variety of Python packages exist which can be used for GMM such as that at ScikitLearn and Pomegranate. Explain GMM, EM algorithm etc 
These would be easilty comptabile with a simple ewb application written in Flask and Python. 

Therefore in this study I implemeted a python-based approach to GMM of rat and human data from the IAP protein.
With the assumption that this would form the backend of future software for automated modelling of Gaussian peaks within all cIMMS data. 

