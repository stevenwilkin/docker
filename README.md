# Dockerized environment

This environment is essentially a combination of the [Nextgen](https://github.com/msri/msri.nextgen/blob/master/docker/README.md) and [Locomotive](https://github.com/msri/msri.locomotive/blob/docker/docker/README.md) Docker environments and assumes the following repositories are checked out at the same level on the file system:

* https://github.com/msri/msri.nextgen
* https://github.com/msri/msri.locomotive
* https://github.com/msri/locomotive

The steps here are derived from the existing Docker `README`s.


## Nextgen


### Download symmetric encryption key

Download the archived encryption key from [the Mac Development Setup wiki page](https://wiki.msri.org/display/MSRI/Mac+-+Development+Setup) and save it to `docker/msri_nextgen.tar` withing the Nextgen repository.


### Import MySQL database

	docker-compose up -d mysql
	docker cp ~/tmp/msri_nextgen.sql $(docker-compose ps -q mysql):/tmp/msri_nextgen.sql
	docker-compose exec mysql bash -c 'mysql msri_nextgen < /tmp/msri_nextgen.sql'
	docker-compose exec mysql rm /tmp/msri_nextgen.sql


### Configure Solr

	docker-compose up -d solr
	docker cp ../msri.nextgen/solr $(docker-compose ps -q solr):/var/lib/solr/
	docker-compose restart solr

### Re-index Solr

	docker-compose build nextgen
	docker-compose up -d nextgen
	docker-compose exec nextgen rails sunspot:reindex


## Locomotive


### Import MongoDB database

Assumes a dump of the production MongoDB is located at `~/tmp/locomotiveapp_production`.

	docker-compose up -d mongo
	docker cp ~/tmp/locomotiveapp_production $(docker-compose ps -q mongo):/tmp/locomotiveapp_production
	docker-compose exec mongo bash -c 'mongorestore --db locomotiveapp_dev --drop /tmp/locomotiveapp_production'
	docker-compose exec mongo rm -rf /tmp/locomotiveapp_production


### Start everything else

	docker-compose up


## Configure Locomotive

Login to the Locomotive admin at [http://localhost:8080/locomotive](http://localhost:8080/locomotive) using your production credentials.


### Allow access from localhost

Browse to "General Settings" at [http://localhost:8080/locomotive/msri/current_site/edit](http://localhost:8080/locomotive/msri/current_site/edit).

In the "Domains" section of the "Access Points" tab add an entry for "localhost" and click the "Save" button. An IP address won't be accepted.


### Set the application environment

Under the "Models" section of the main navigation click on "Settings" and from there click on "environment". Change the value to "development" and click the "Save" button.


## Configure Wagon

Retrieve your API key from [the account page](http://localhost:8080/locomotive/my_account/edit) of the Locomotive admin.

Create `config/deploy.yml` within the Wagon repository with the following format:

	development:
	  host: http://localhost:8080
	  handle: msri
	  email: <EMAIL>
	  api_key: <API KEY>


## Deploy the layout and templates to Locomotive

Ensure the dependencies are installed and from the Wagon repository run:

	bundle exec wagon deploy development



Full environment is available at [http://localhost:8080](http://localhost:8080)