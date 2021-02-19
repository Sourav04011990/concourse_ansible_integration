fly-login:
	fly \
		--target ci \
		login \
		--concourse-url=https://ci.comcast.net \
		--team-name=cubits \
		--open-browser

fly-set-pipeline:
	fly \
		--target ci \
		set-pipeline \
		--pipeline concourse-ansible-autobahn-example \
		--config ci/pipeline.yml
