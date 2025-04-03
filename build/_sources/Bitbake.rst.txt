#######
Bitbake
#######

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

   REQUIRED_POKY_BBLAYERS_CONF_VESION = "2"

**BBPATH** - acts like the PATH variable but adds a directory to the search list for the metadata files
You can either explicitly set it in the command line using ``BBPATH="<projectdirectory>"`` or
you can set in implicitly using a config file like above.

**BBFILES** - List of file for Bitbake to use to build software

**BBFILE_COLLECTIONS** - Lists the names of configured layers. These names are used to find the other BBFILE_* variables. Typically, each layer
appends its name to this variable in its *conf/layer.conf* file.

**BBFILE_PATTERN_\*\*** - Variable that expands to match files from
BBFILES in a particular layer. This variable is used in the *conf/layer.conf* file and must be suffixed with the name of the specific layer(e.g BBFILE_PATTERN_emenlow)

.. note:: The meaning of "^${LAYERDIR}/":
   ^ tells bitbake to go down 2 layer 

