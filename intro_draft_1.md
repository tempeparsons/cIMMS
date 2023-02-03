Introduction. 

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

