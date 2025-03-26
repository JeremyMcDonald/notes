######################################
Embedded Linux Using The Yocto Project
######################################

*Bitbake is a generic task excecution engine that allows shell and Python tasks to be run efficiently and in parallel while*
*working within complex inter-task dependancy constraints* - From the bitbake docs

Brief update about layers
-------------------------
* OpenEmbedded Core: This is inside the **meta** directory
* Poky distrobution: This is inside the **meta-poky** directory
* Yocto Project reference BSP: This is inside the **meta-yocto-bsp** directory

Every layer contains at least a **conf/layer.conf** 

.. code-block:: bash
   :linenos:

   #This is a simple example of a layer.conf
   #We have a conf and a classes directory, add to BBPATH
   BBPATH =. "{$LAYERDIR}:"
   
   # We have recipes-* directories, add to BBFILES
   BBFILES += "${LAYERDIR}/recipes-\*/\*/\*.bb \
               ${LAYERDIR}/recipes-\*/\*/\*.bbappend"

   BBFILE_COLLECTIONS += "yocto"
   BBFILE_PATTERN_yocto = "^${LAYERDIR}/"
   BBFILE_PRIORITY_yocto = "5"

   # This should only be incremented on significant changes that will
   # cause compatability issues with other layers
   LAYERSERIES_yocto = "3"

   LAYERDEPENDS_yocto = "core"

   REQUIRED_POKY_BBLAYERS_CONF_VESION = "2"``

BBPATH acts like the PATH variable but adds a directory to the search list for the metadata files


