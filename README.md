# Hybrid-TotalSegmentator-and-nnU-Net-for-Craniofacial-Segmentation
<p align="center"> 
  <img width="468" height="269" alt="image" src="https://github.com/user-attachments/assets/af85e08c-86d7-4f13-925b-9940a145cb88" /> 
</p> 
Develop a DL framework based on nnU-Net v2, together with TotalSegmentator, to automatically segment the mandible and cranial base in CBCT images. This non-manual labeling and segmentation method will be the essential foundation for future automatic diagnosis of dentofacial deformities.

#### OVERVIEW
This repository contains the methodology for developing and validating an AI-based framework for automatic mandibular segmentation from CBCT images. The pipeline combines automated ground truth generation (via TotalSegmentator), a fully self-configuring nnU-Net v2 architecture, and a Tversky-loss-driven training scheme optimized for the class imbalance typical of craniofacial CBCT data.
#### Key highlights:
- 270 subjects, 150,000+ CBCT slices (16-bit grayscale)
- Automated GT labeling pipeline using TotalSegmentator with manual QC
- 3D nnU-Net v2 architecture with 6-stage encoder/5-stage decoder
- Tversky loss (α = 0.3, β = 0.7) tuned to favor recall for complete mandibular coverage
- Evaluated with Dice Coefficient, Precision/Recall, and Hausdorff Distance (HD95)
#### Dataset
- Acquisition
CBCT scans were acquired from patients referred to the Department of Oral and Maxillofacial Surgery for orthognathic surgery between 2013–2025, using one of three scanner units: Carestream 9300; Carestream 9600; i-CAT FLX. All scans followed standardized imaging protocols with patients positioned upright to preserve natural anatomical relationships.
- Inclusion & Exclusion Criteria
Inclusion: Males/females, ages 18–45; diagnosis of maxillary/mandibular excess or deficiency, anterior open bite, deep bite, or unilateral/bilateral transverse maxillary deficiency; complete mandible visualization (condyle to symphysis).
Exclusion: Significant motion artifacts; extensive uncorrectable metal artifacts (implants/orthodontic hardware); prior maxillofacial surgery altering cranial base/mandibular anatomy; craniofacial developmental syndromes or severe systemic bone disease.
After filtering, 270 subjects were retained for model development and validation.
- Dataset Composition & Splits
Subject identifiers (name, acquisition date, medical record number, etc.) were removed prior to data handoff to preserve anonymity.
Split Subjects Percentage: Training 74.1%(200) / Validation 13.0%(35) / Test 13.0%(35).
#### Preprocessing Pipeline
A six-step preprocessing pipeline standardized all CBCT volumes prior to training:
1. Normalization: Z-score standardization: $I_norm = (I - μ) / σ$
2. Gaussian Filtering: σ = 0.5 mm, balances noise suppression vs. boundary preservation
3. Resampling: Trilinear interpolation to isotropic 0.4 × 0.4 × 0.4 mm³ voxel spacing
4. Registration: Rigid transformation to a common coordinate frame via mutual information maximization
5. Windowing: HU-equivalent window center = 400, width = 1800 (bone emphasis)
6. Data Augmentation: Random rotation (±15°, all axes), horizontal/sagittal flipping, random scaling (0.9–1.1×), elastic deformation
#### Ground Truth Generation
Manual annotation of 270 volumes was impractical given clinician time constraints and inter-observer variability. Ground truth labels were instead generated automatically using TotalSegmentator, an open-source nnU-Net-based segmentation tool trained on 1,204 clinical CT exams and capable of segmenting 104 anatomical structures (27 organs, 59 bones, 10 muscles, 8 vessels) with a reported Dice Score of 0.943.
- Label Inference Process: 
1. Format conversion: DICOM → NIfTI (better compatibility with Python tooling, e.g., SimpleITK)
2. Weight acquisition: Pre-trained TotalSegmentator weights loaded for structure identification
3. Automated labeling: Region of interest restricted to mandible and cranial base
- 0 = Background;
  1 = Mandible;
  2 = Cranial base;
  Teeth (maxillary and mandibular) excluded from segmentation
#### Quality Control
All auto-generated labels were reviewed in ITK-SNAP for anatomical accuracy and boundary correctness. Cases with visible errors were manually corrected before being finalized as training labels.
#### Model Architecture
##### nnU-Net v2
nnU-Net is a self-configuring segmentation framework that automatically adapts preprocessing, network architecture, and training parameters to dataset-specific properties (voxel spacing, intensity distribution, class imbalance).
##### Network Details
Type: 3D U-shaped encoder-decoder with skip connections \
Encoder: 6 resolution stages, each with two 3×3×3 convolutions + normalization + Leaky ReLU; Channel progression: [32, 64, 128, 256, 320, 320] \
Downsampling via strided convolution (no max-pooling): [2,2,2] at intermediate stages, anisotropic [1,2,2] at the deepest level \
Bottleneck: 320 channels, same conv structure as encoder blocks \
Decoder: 5 upsampling stages using transposed convolutions + skip-connection concatenation, followed by two 3×3×3 conv layers (Instance Norm + Leaky ReLU) per stage \
Input patch size: 112 × 160 × 128 voxels \
Voxel spacing: 0.3 mm isotropic \
Batch size: 2
##### Loss Function
Given severe class imbalance in craniofacial CBCT (small target structure relative to volume), Tversky loss was used instead of standard Dice/cross-entropy loss:
$L_Tversky = 1 − TP / (TP + α·FP + β·FN)$ \
With α = 0.3, β = 0.7 — prioritizing recall to minimize under-segmentation of the mandible.
##### Training Configuration
- Optimizer: Adam (β₁ = 0.9, β₂ = 0.999)
- Initial learning rate: 0.01, polynomial decay: $lr = lr₀ × (1 − epoch/epoch_total)^0.9$
- Batch size: 2
- Epochs: 700
- Weight decay: $3 × 10^-5$
- Patch size: 112 × 160 × 128 (auto-configured)
##### Training strategy
- 1-fold cross-validation (fold 0) for preliminary hyperparameter tuning
- Mixed precision training via PyTorch native AMP
- Early stopping monitored on validation Dice (patience = 50 epochs); full 700 epochs completed in this study.
#### Citation
##### If you use Hybrid TotalSegmentator and nnU-Net in your research for non-manual labeling and segmentation, please cite:
```bibtex
@ARTICLE{
  author={Jin, Congjun and Issa, Raja Raymond A, and Widmer, Charles G},
  title={Hybrid TotalSegmentator and nnU-Net Framework for Craniofacial Segmentation},
  year={2026},
  volume={},
  number={},
  pages={1-21},
  keywords={Deep learning, nnU-Net v2, TotalSegmentator, Craniofacial Bone},
}
```
#### Acknowledgements
### This research was supported by a University of Florida University Research Investment (URI) Award. We acknowledge the University of Florida Integrated Data Repository (IDR) and the UF Health Office of the Chief Data Officer for providing the analytic data set for this project.
