# Repo Management

The goal is to simplify maintaining minimal device manifests for hybris Hardware Adaptations (HA).

We need a default.xml manifest to provide:

1. a minimal set of standard (unmodified) android git repos which are dependencies for the HA
2. the set of 'standard' hybris git repos which are common to all hybris device ports
3. unmodified device specific git repos
4. modified device specific git repos

By default all git repos in the upstream manifest are dropped.

Since this is a little extreme we create two keep.pathlist files which say which ones to keep. There's a common one and a device specific one. This satisfies points 1 and 3.

Next we have a common default.xml.template which contains these sections:

* ```<include>``` of the original base manifest as a starting point
* the hybris remote name (typically github but possibly a private git server)
* a placeholder to be automatically populated with all the manifest paths not mentioned in the keep.pathlist
* a list of hybris-modified repos (bionic etc) for point 2
* ```<include>``` of the ha.xml repo for device specific repos and overrides.

Finally a device-specific ha.xml manifest allows additions and overrides to both the base and common manifests without needing to edit them.

## Pathlists

The pathlists are simply that - a list of directory paths relative to the base dir. eg system/extras
Any project fully matching that path is kept.

In addition you can match any project using a regular expression by starting the path with "re: "
eg: re: ^hardware/.*
will keep all projects in the hardware/ directory tree.

## A word about versions
It's important to use the correct revision of each git respository. The CM and AOSP manifests will likely use the ```<default>``` element to set the revision for the unmodified repos.

The modified hybris repos will need a similar revision to match the rest of the droid build or will use the master branch.

The create_default command takes the hybris revision and inserts it into the default.xml file (not ha.xml - that must be managed manually)

## ha.xml manifest and overrides

The ha.xml should be a full manifest in its own right.

See ```repo help manifest``` for details but additional repos can be placed in the ha.xml using ```<project>``` and overrides are done using ```<remove-project>``` followed by ```<project>```.


## Get Started

* Create an empty repo called hybris-manifest-$DEVICE eg hybris-manifest-mako
* Add an empty initial commit
* Add the hybris-manifest git submodule
* Find a suitable CM or AOSP manifest for your device and add it into the repo
* Run: ```hybris-manifest/create_default <your manifest file> <hybris-revision>```
* Edit the ha.xml to point to your device-specific droid-hal-$DEVICE file
* commit the default.xml to your repo

To modify the manifest, update the pathlists or submodule as needed and re run the create_default command

	
	DEVICE=i9305
	mkdir hybris-manifest-$DEVICE
	cd hybris-manifest-$DEVICE
    git init .
	git commit --allow-empty -sm"Initial commit"
	git submodule add  https://github.com/mer-hybris/hybris-manifest.git hybris-manifest
	git commit -sam"Added hybris-manifest submodule"

	# Create cm-10.1.3.xml - this is the CM default manifest

	# Now create a default.xml using templates and the correct branch
    # of the various common repos
	hybris-manifest/create_default cm-10.1.3.xml hybris-10.1.3

	# Edit ha.xml to minimise it - it pulls in all kinds of stuff for
    # many devices by default

	git add *xml keep.pathlist
	git commit -sam"Created manifest for $DEVICE"

# To Build

Follow the HADK but use your new manifest:

	repo init -u https://github.com/mer-hybris/hybris-manifest-$DEVICE.git
