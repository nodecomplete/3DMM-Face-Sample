# 3DMM-Face-Sample

>Note: This project was done to help me understand what 3DMM models are and how they are used with face reconstruction DNNs. While it can be used to create a 3DMM of a human face, it is unlikely it will be a substiture for a production quality 3DMM that would be useful in a research or commercial use. Consider the following to be of education value only.

![alt text](https://github.com/nodecomplete/3DMM-Face-Sample/blob/master/FaceMorph/Faces.jpg)

**Introduction** 

Deep Neural Networks (DNNs) that take as an input a 2D image of a human face and output a 3D mesh (e.g. Microsoft's [Deep-3D-Face-Reconstruction](https://github.com/microsoft/Deep3DFaceReconstruction)) often incorporate a 3D Morphable Face Model (3DMM) in their pipeline. One such example is [Basel](https://faces.dmi.unibas.ch/bfm/) as used by [Deep-3D-Face-Reconstruction](https://github.com/microsoft/Deep3DFaceReconstruction). When encountering such a DNN for the first time, one might expect that the mesh, which consists of vertices and indices, is perhaps output directly by the final layer of the DNN (the basel face mesh used in Deep3DFaceReconstruction has over 70,000 triangles). This is turns out not to be true however. Below we look at what exactly a 3DMM is, and how it is used in face recosntruction/mesh creation DNNs.

**Discussion** 

3 3DMM is created by scanning multiple faces e.g. using a 


Each mesh is represented as a series of shape vectors:

<img src="https://render.githubusercontent.com/render/math?math={S^0,%20S^1%20%20%20...%20%20%20S^{m-1} }">

Each shape is represented as a column vector with n vertices:

<img src="https://render.githubusercontent.com/render/math?math={S^{i} =  \begin{bmatrix}   x^{i}_{0}   \\ \vdots  \\x^{i}_{n-1}\\y^{i}_{0}   \\ \vdots  \\y^{i}_{n-1}\\z^{i}_{0}   \\ \vdots  \\z^{i}_{n-1}  \end{bmatrix}  \in  \Re ^{3n}}">

where the mean/average mesh is defined as: 

<img src="https://render.githubusercontent.com/render/math?math={\overline{S} =   \frac{1}{m} \sum_{i=0}^{m-1} S^i %20}">

If we subtract the average from each mesh 

<img src="https://render.githubusercontent.com/render/math?math={ \bigtriangleup S^i=(S^i -\overline{S}) }">

then new models are expressed as the mean plus a linear combination of the m shape vectors:

<img src="https://render.githubusercontent.com/render/math?math={  S^{model}  =\overline{S}%2B\sum_{i=0}^{m-1} \alpha^{i}  \bigtriangleup S^i}">

The dimensionality of the system is m but we can use [Principle Component Analysis](https://en.wikipedia.org/wiki/Principal_component_analysis) to determine a new set of basis vectors for the system such that the most dominant components are included first and the rest can be ignored. i.e.

<img src="https://render.githubusercontent.com/render/math?math={  S^{model}  =\overline{S}%2B\sum_{i=0}^{k-1} \alpha^{i}  \E^i}">

where k is the new dimension (hopefully significantly smaller than m) and E represent the Eigenvectors returned from the [Singular Value Decomposition](https://en.wikipedia.org/wiki/Singular_value_decomposition) (SVD) method that we use to perform the Principle Component Analysis.

In the demo application, the Meshes folder includes 204 face meshes and so we have m = 204. After performing SVD on these meshes, only the first 20 are included and the rest are discarded, so we have k=20. In the image below, the 20 sliders are used to control the scalar multipliers (presented as alpha in the previous equation):

<img src="https://render.githubusercontent.com/render/math?math={\alpha^{i},  %20%20%20%20  \0 \leq  i < k}">
 
The image below shows theses scalars set to zero, so the image displayed is the average/mean face. The second image shows the change in the mesh when the first slider (first principle component) is non-zero.
 

![alt text](https://github.com/nodecomplete/3DMM-Face-Sample/blob/master/FaceMorph/ScreenShot2.jpg)

[comment]: <> (<img src="https://render.githubusercontent.com/render/math?math={\bigtriangleup S=\begin{bmatrix} \vdots   \\ {\bigtriangleup S^0  \bigtriangleup S^1  \ldots  \bigtriangleup S^{m-1}} \\ {\vdots  } \\ \end{bmatrix}}">)
 
![alt text](https://github.com/nodecomplete/3DMM-Face-Sample/blob/master/FaceMorph/ScreenShot.jpg)

The following conclusions can be drawn from the above analysis:

&nbsp;&nbsp;1) A DNN that produces a face mesh as its output just needs to return a vector of scalars that are then applied to the 3DMM to produce the actual mesh.

&nbsp;&nbsp;<img src="https://render.githubusercontent.com/render/math?math={\alpha  =  \begin{bmatrix}   \alpha_{0}   \\ \vdots  \\\alpha_{k-1}\\   \end{bmatrix}  }">

&nbsp;&nbsp;2) A 3DMM of a human face consists of the following:

- k eigenvectors, each of length n where n is the number of vertices of the face mesh.
 
- the average/mean face mesh also of dimension n.
 
- indices that will apply to vertices to form the actual triangles.
 
 
**Building the software** 

1) The software was built using [Visual Studio 2019 community edition](https://visualstudio.microsoft.com/downloads/). Be sure to install the MFC module.
2) It requires DirectX 9 [June 2010 DirectX SDK ](https://www.microsoft.com/en-nz/download/details.aspx?id=6812)
3) The software uses the [Eigen library](http://eigen.tuxfamily.org/index.php?title=Main_Page) to perform the SVD. To install Eigen, navigate to the C drive and clone it from github:

&nbsp;&nbsp;&nbsp;&nbsp;`cd C:\`

&nbsp;&nbsp;&nbsp;&nbsp;`git clone https://gitlab.com/libeigen/eigen.git`
 
 
**Running the software** 

When executed, the EXE performs the following:
1) loads the sample nmeshes from the 'meshes' folder 
2) Calculates the average mesh by adding all vertices and dividing by the total number of meshes
3) subtracts the average mesh from each of the meshes
4) Calculates the eigenvectors using the Eigen libraries 
5) Creates and renders a mesh on-the-fly created using the following equation where the scalar paramters are dynmically updated from the slider controls.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://render.githubusercontent.com/render/math?math={  S^{model}  =\overline{S}%2B\sum_{i=0}^{k-1} \alpha^{i}  \E^i}">


**Additional Notes**

1) The software won't display the main dialog box until the above steps are complete, so it may appear not to start up right away.
2) Normally the sample meshes (location in the 'meshes' folder) used to create the model are scanned from real faces. In this case it wasn't practical to do this, and the meshes were createed using [Deep-3D-Face-Reconstruction](https://github.com/microsoft/Deep3DFaceReconstruction). The images from which the meshes were created were aligned (with respect to eyes, nose...) to produce better results. See [here](https://www.youtube.com/watch?v=OaCmD08xxGw) why alignment of the meshes is important.
3) Production quality 3D Morphable Models of human faces typically seperate shape from facial expressions and have a set of eigenvectors for both. Some also include a third set for texture of the face. 
4) Fianlly. I have not attempted to optimize the eprformance (via threading, vectorizeation, cache coherency etc) and I oplogize for using a legacy version of DirectX!!



