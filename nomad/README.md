# Nomad
A Sample art of work on Nomad

## Architecture Diagram

![nomad-architecture-region-a5b20915](https://user-images.githubusercontent.com/8342133/34456790-27df08d0-edc6-11e7-9f4a-62679f6e7a3d.png)

For multiple data center

![nomad-architecture-global-a8f14b78](https://user-images.githubusercontent.com/8342133/34456799-4246d536-edc6-11e7-9ff8-4663bb55f463.png)

## Vault Config File

Sample nomad.conf file

````
bind_addr = "0.0.0.0" # the default

data_dir  = "/var/lib/nomad"

advertise {
  # Defaults to the node's hostname. If the hostname resolves to a loopback
  # address you must manually configure advertise addresses.
  http = "1.2.3.4"
  rpc  = "1.2.3.4"
  serf = "1.2.3.4:5648" # non-default ports may be specified
}

server {
  enabled          = true
  bootstrap_expect = 3
}

client {
  enabled       = true
  network_speed = 10
  options {
    "driver.raw_exec.enable" = "1"
  }
}

consul {
  address = "1.2.3.4:8500"
}
````
## Setup

* [Installation](#installation)
* [Jobs](#jobs)
* [Dev Mode](#dev-mode)
* [UI](#ui)

#### Installation

Download the binary from https://www.nomadproject.io/downloads.html

````
$ chmod +x nomad
$ mv vault /usr/local/bin/
````

#### Jobs

````
//Sample Job File

# There can only be a single job definition per file.
# Create a job with ID and Name 'example'
job "helloapp" {
	# Run the job in the global region, which is the default.
	# region = "global"

	# Specify the datacenters within the region this job can run in.
	datacenters = ["dc1"]

	# Service type jobs optimize for long-lived services. This is
	# the default but we can change to batch for short-lived tasks.
	# type = "service"

	# Priority controls our access to resources and scheduling priority.
	# This can be 1 to 100, inclusively, and defaults to 50.
	# priority = 50

	# Restrict our job to only linux. We can specify multiple
	# constraints as needed.
	constraint {
		attribute = "${attr.kernel.name}"
		value = "linux"
	}

	# Configure the job to do rolling updates
	update {
		# Stagger updates every 10 seconds
		stagger = "10s"

		# Update a single task at a time
		max_parallel = 1
	}

	# Create a 'cache' group. Each task in the group will be
	# scheduled onto the same machine.
	group "hello" {
		# Control the number of instances of this group.
		# Defaults to 1
		count = 1

		# Configure the restart policy for the task group. If not provided, a
		# default is used based on the job type.
		restart {
			# The number of attempts to run the job within the specified interval.
			attempts = 2
			interval = "1m"

			# A delay between a task failing and a restart occurring.
			delay = "10s"

			# Mode controls what happens when a task has restarted "attempts"
			# times within the interval. "delay" mode delays the next restart
			# till the next interval. "fail" mode does not restart the task if
			# "attempts" has been hit within the interval.
			mode = "fail"
		}

		# Define a task to run
		task "hello" {
			# Use Docker to run the task.
			driver = "docker"

			# Configure Docker driver with the image
			config {
                          image = "gerlacdt/helloapp:v0.1.0"
                          port_map {
                            http = 8080
                          }
                        }
			service {
				name = "${TASKGROUP}-service"
				tags = ["global", "hello", "urlprefix-hello.internal/"]
				port = "http"
				check {
				  name = "alive"
				  type = "http"
				  interval = "10s"
				  timeout = "3s"
				  path = "/health"
				}
			}

			# We must specify the resources required for
			# this task to ensure it runs on a machine with
			# enough capacity.
			resources {
				cpu = 500 # 500 MHz
				memory = 128 # 128MB
				network {
					mbits = 1
					port "http" {
					}
				}
			}

			# The artifact block can be specified one or more times to download
			# artifacts prior to the task being started. This is convenient for
			# shipping configs or data needed by the task.
			# artifact {
			#	  source = "http://foo.com/artifact.tar.gz"
			#	  options {
			#	      checksum = "md5:c4aa853ad2215426eb7d70a21922e794"
			#     }
			# }

			# Specify configuration related to log rotation
			logs {
			    max_files = 10
			    max_file_size = 15
			}

			# Controls the timeout between signalling a task it will be killed
			# and killing the task. If not set a default is used.
			kill_timeout = "10s"
		}
	}
}

````

#### Dev mode

````
$ sudo nomad agent -dev
````

#### UI

````
//Display UI
$ nomad ui
````

## License

MIT License

## References

