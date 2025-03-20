This repo contains scripts to generate the impact model data for generated sites.

It requires both the site_generation and bridges modules to be installed, both of which have been included in the
[python_modules](./python_modules) directory. Further, a [dockerfile](./dockerfile) can be used to generate a docker
image that will automatically install the required modules.

There have been some issues with the data downloads, so the data needed to run the run scripts has been included in the upload to the aws bucket.

If you want to run this for a new country, data for that country will attempt to download, and as of July 2024,
the data download for any African country should work as long as you have also downloaded the shared_data folder. As a note, the only destination data that is generated
by this module are the major roads and semi-dense urban datasets. Schools and hospitals will need to be downloaded and 
put in the correct directory, ie bridges_data/shared_data/destination_data/{country}/{country}_cleaned_destinations.


You would then need to add that country to the country list in the various run_scripts.py files:

[no_order_1_less_than_500m_site_generation/run_script.py](./no_order_1_less_than_500m_site_generation/run_script.py)\
[no_order_1_less_than_500m_with_top_sites/run_script.py](./no_order_1_less_than_500m_with_top_sites/run_script.py)\
[no_order_1_site_generation/run_script.py](./no_order_1_site_generation/run_script.py)\
[no_order_1_with_top_sites/run_script.py](./no_order_1_with_top_sites/run_script.py)

You will further need to add a configuration file for that country to each of the configuration file directories:

[no_order_1_less_than_500m_site_generation/configuration_files](./no_order_1_less_than_500m_site_generation/configuration_files)\
[no_order_1_less_than_500m_with_top_sites/configuration_files](./no_order_1_less_than_500m_with_top_sites/configuration_files)\
[no_order_1_site_generation/configuration_files](./no_order_1_site_generation/configuration_files)\
[no_order_1_with_top_sites/configuration_files](./no_order_1_with_top_sites/configuration_files)

You only need to add a global_config.yml file in each that has the country name, and continent.
An example for Rwanda is below

[Rwanda Configuration file example.](./no_order_1_with_top_sites/configuration_files/rwanda/global_config.yml)


Finally, the run_scripts should always be run in order: ..._site_generation, ..._with_top_sites.
eg to generate the data for no order 1 waterways, first run:

```
python ./no_order_1_site_generation/run_script.py
```
then run
```
python ./no_order_1_with_top_sites/run_script.py
```

The outputs will populate in the appropriate directory, eg:

[no_order_1_with_top_sites/model_outputs](./no_order_1_with_top_sites/model_outputs)

### Adding Additional Destinations
To include a destination type, you must have a cleaned destination type parquet file saved in 

```
shared_data/destination_data/{region}/{region}_cleaned_destinations/{destination_type}.parquet
```
where region and destination_type are replaced with the appropriate titles. This should be done for all regions that you 
would like to use this data for.

EG to use all education facilities for Rwanda, the file all_education_facilities.parquet should exist with the following path:
```
shared_data/destination_data/rwanda/rwanda_cleaned_destinations/all_education_facilities.parquet
```
Next, you need to add that destination type to the generic/global_config.yml and to generic/graph_generation.yml in the with top sites 
directories (it is not necessary to add the destination to the site generation configuration files).

Finally, when you run run_script.py, edit the country_list to only include countries that include the above dataset.

### Docker
There is a working docker file in the base repo. 

With docker installed, and in the bridges_data directory (where this readme is) run
```
docker build . -t bridges
```
Depending on your setup you may need root privileges.

As a note, the dockerfile excludes most of the subdirectories in bridges_data. 
I suggest that you simply give docker access to these directories.
This behavior can be changed by deleting (or commenting out) the directory names
from the .dockerignore file.

The purpose of excluding the data subdirectories is to avoid
copying the data to the docker image.
The these directories can later be attached to the docker container running that image, which will
give easier access to the output and input of the container.

To run a container with those directories mounted, from the bridges_data directory run
```
docker run --cpus=0 \
          -v ./no_order_1_less_than_500m_site_generation:/bridges_data/no_order_1_less_than_500m_site_generation \
          -v ./no_order_1_less_than_500m_with_top_sites:/bridges_data/no_order_1_less_than_500m_with_top_sites \
          -v ./no_order_1_site_generation:/bridges_data/no_order_1_site_generation \
          -v ./no_order_1_with_top_sites:/bridges_data/no_order_1_with_top_sites \
          -v ./shared_data:/bridges_data/shared_data \
          -it \
          bridges \
          bash
```


The options in the above call are as follows:
```
--cpus=0 allows docker to use all of the computers cpus. If you wish to limit the number of cpus to n, use -cpus=n 
-v ./{dirctory}:/bridges_data/{dirctory} mounts this directory to the appropriate disk directory
                                         (assuming this has been run from the bridges_data directory).
-it makes an interactive pseudo terminal
```

Finally, the run scripts can be run as discussed above.
