## Prepare filePaths.tsv

```
cd /bia-idr/S-BIAD2282/REMEDI4ALL_01
find * -maxdepth 3 -type f -name "METADATA.ome.xml"
```

## Import
```
# Create new screen
omero obj new Screen name="idr0171-kuzikov-remedi4all/screenA"

# Import metadata.xml
echo "loci.formats.in.OMEXMLReader" > /tmp/reader.txt
/opt/omero/server/OMERO.server/bin/omero import -l /tmp/reader.txt --bulk idr0171-screenA-bulk.yml --file /tmp/171.log

# Create conda env
micromamba env create -n 171 conda-forge::omero-py
micromamba activate 171
# install cli-render plugin too
pip install omero-cli-render 

# Use omero-cli-zarr extinfo branch to set ExternalInfo
git clone https://github.com/dominikl/omero-cli-zarr.git
cd omero-cli-zarr
git checkout extinfo
pip install -e .
pip install ome-zarr==0.11.1 zarr==2.18.7

# Set ExternalInfo
for i in `cat 171.log`; do omero zarr extinfo --set $i; done

# Do the rendering/thumbnails
parallel -j 8 --eta --progress --delay 3 omero render test --thumb :::: 171.log
```


