### Centos 7 Atlas Makefile Patch

* Replace LIBRARIES += cblas atlas in Makefile:368 with LIBRARIES += tatlas satlas
* Remember to check in Makefile.config WITH_PYTHON_LAYER := 1 and check that system paths for python are used (and not anaconda or other)

Must build caffe: http://caffe.berkeleyvision.org/installation.html#compilation
* Must build caffe within *py-faster-rcnn project:  https://github.com/BVLC/caffe/issues/4619
 * make all -j16
 * make test -j16 # optional
 * make runtest -16 # optional

Add "make pycaffe " after that
Enter $FRCN_ROOT/lib and do 'make' : https://github.com/rbgirshick/fast-rcnn/issues/77

### Centos 7 Dependencies. Only easydict is not in repos

Packages: wget scikit* protobuf-devel protobuf protobuf-python lmdb-devel hdf5-devel leveldb-devel snappy snappy-devel Cython easydict(pip)

* Add as 1st import in train_net.py and test_net.py if no X server is used. (If there is no graphical interface on the server) / Añadir como el primer import en train_net.py
> import matplotlib as mpl
> mpl.use("Agg")

* Actualizar rutas de descarga de scripts, segun URLs en https://github.com/rbgirshick/py-faster-rcnn/blob/master/data/scripts/ (Solo si no se clona caffe desde el repo original)
* Ejecutar el script fetch_faster y fetch_imagenet_models.sh en /data/scripts/: 
> sh data/scripts/fetch_faster_rcnn_models.sh

### Modificaciones por uso de TeslaM2090 (compute capability 2)

* Modificar lib/fast_rcnn/config.py : __C.USE_GPU_NMS = False desde True 
* Parchar codigo: https://github.com/BVLC/caffe/pull/5002/files
* Reducir el batch size (< 100): en config.py (soluciona -> cudaSuccess (9 vs. 0)  invalid configuration argument :)

### Problemas del codigo de Python

* mtrand.pyx: 'numpy.float64' object cannot be interpreted as an index -> https://github.com/rbgirshick/py-faster-rcnn/issues/481 

### How to Run
> ./experiments/scripts/tsd_full_faster_rcnn_end2end.sh 0 ZF tsd_full

### Propio dataset
* crear carpeta data en cualquier lugar (despues hay que dejarla en data/proyecto/)
* copiar txt o crearlo a partir de csv. Agruparlos y correr:
> for i in $(ls Annotations/ | sed 's/\.csv//g' | sed 's/GT-//g'); do for j in $(ls Images/$i | grep -v csv); do echo "$( echo $i/$j | sed 's/\.ppm//g')" >> ImageSets/train.txt   ;done ; done 

* cambiar annotations csv por txts 
> for i in $(ls); do tmp=$(echo $i | sed 's/\.csv//g' | sed 's/GT-//g'); for j in $(cat $i ); do echo "$( echo $tmp/$j )" >> gt.txt  ;done ; done

* en /lib/datasets/ copiar pascal_voc.py, renombrar archivo y clase a tsd_full.py. Ajustar las classes en self._classes. Revisar self._image_ext 

* en models/tsd/ZF/faster_rcnn_end2end/train.prototxt hay q cambiar
 > layer {   name: 'roi-data' , en param_str: "'num_classes':  al numero de nuevas classes (classes + 1)
 > layer {   name: "cls_score", en num_output: 
 > layer {   name: "bbox_pred", num_output: (classes + 1) * 4


### Error si se cambia el scale al llamar a generate_anchors
>  File "/home/ml/w-py-faster-rcnn/tools/../lib/rpn/proposal_layer.py", line 141, in forward
>    proposals = proposals[order, :]
> IndexError: index 0 is out of bounds for axis 0 with size 0

* Se pudo modificar en lib/fast_rcnn/config.py 
> __C.TRAIN.RPN_MIN_SIZE = 8

### Mejorar el dataset 
* Opcion 1: Tomar data desde http://btsd.ethz.ch/shareddata/BelgiumTS/Annotations/camera0*.tar  y desglosar/transformar lo que sirva
* Opcion 2: Dado los problemas para que detecte señales cuando la imagens es mayor que ~ 255x255, se multiplicaran por 10 las imagenes grandes
del GTSRD

* Script de creacion de imagenes en cache de faster rcnn no soporta que nombre de imagenes tengan un . aparte del que ya tienen para la extension. Ej: image.1234.ppm will raise error
