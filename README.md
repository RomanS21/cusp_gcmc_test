# cusp_gcmc_test
A Simple repo to explore the CUSP ecosystem to run GCMC for MOFs

Contents:
GCMC: 
  GCMC is run using the CUSP kUPS code, for the example RUBTAK/CO2 adsoroption case. Producing adsoption isotherms and Isosteric heat of adsorption plots.
  The next step will be to reweight and rerun with a foundation MACE model. Lookinng to benchmark the accuracy of the MACE model against the UFF/TraPPE potential. This is not expected to be a good match.
  Resultant values will be used to benchmark the accuracy of a smaller distilled model. The same reweighting will be done with the distilled MACE model expecting to recover the same poor match as with the foundation model
  (details of distilation discussed below)
Widom: 
  Widom insertion for RUBTAK and CO2 run using the kUPS code as per example for teh UFF/TraPPE potential. The standalone CUSP widom package is also used to calculate henry coefficients for MACE type models and the MACE distilled model.
Model Distilation:
  The working plan is to use data generated in the GCMC and Widom with the UFF/TraPPE potential, labeled with a foundation model to train a smaller lightweight MACE model. A committee of these models will be used to propogate the same insertion/GCMC methods gennerating more data to further train the distilled model. The distilled model will be benchmarked against the foundation model and the original GCMC/Widom runs.
  
  This is a time intensive task that may not be complete by the time this repo is seen so I will highlight design choices here:
  The goal will be to create a very small and lightweight model. The primary parameters I will relax compared to foundation models are the r_max, number of channels and max L. 
  r_max - this directly reduces the size of the neighbour list that must be considered/updated/calculated in update moves. Reducing it not only speeds up the model evaluation (fewer aggregated properties), but it also allows GCMC codes to potentially save of postion updates/evaluation for particles that are outside the r_max distance. This would significantly speed up the GCMC code with thousands of evaluations for large numbers of materials. 
  The reduction in number of channels is a direct reduction in the number of parameters that must be evaluated for each particle. This will also speed up the model evaluation and reduce the size of the model. Since this is being tuned to a highly specific task there will be a balance point where we will keep accuracy but reduce model evaluation cost. 
  The reduction in the expressiveness of the model works in the same way as higher order features become more expensive to evaluate. Again there will be a balance point and this is likely the last switch to be turned as expressiveness is a key feature of the MACE model.
  To balance all of these cost cutting measures, the relative speed of foundation model evaluation (compared to DFT) will allow us to be in a data rich regime, targetting 1000s to 10,000s of structures to be added to the training set. This ofcourse has an increase to the training cost of the model but this is offset by the same reduction in model size that necessitates an increase in training data.
  
  This will generate a committe (size 3-5 depending on evaluation speed) that can be used to run GCMC and widom calculation. Ideally this model would be small enough to run GCMC and Widom calculations itself. This will allow it to probe model relevant regions of space that might differ to those seen in the UFF/TraPPE potential, correcting any bias introduced by this sampling. Specifically, this will likely sample different loadings as it will have different energetics stemming from a lack of long range/charged interactions. Resultant data will be collected and committee uncertainty calculated and used to sample new points to be evaluated with a foundation model. In GCMC (without MD relaxation) energy is the key property being evaluated, so likely a localised energy committee uncertainty (most likely focused on the inserted CO2) would be most appropriate. This will lead to a new generation of models that can be benchmarked against previously demonstrated large foundation model results to assess convergence. Then used for production runs to compare against experimental data.

  Some limitations of this model choice is that it will be inherently local and unaware of charges. Arhitechtures like MACE-POLAR or something that adds ewald corrections, similar to what is done with the empirical potential, could be appropriate and can be explored. 
  
  For comparison to experiment, the distilled model will inherit the errors of the foundation model, such as potential lack of long range interactions or bad description of atoms in the MOF, as well as reported poor description of CO2 binding. An interesting approach could be to first benchmark and fine tune the foundation model (active learning) with DFT before the distilation. 
