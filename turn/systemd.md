# Overview: Systemd


Systemd is a system and service manager for Linux, it’s widely used as the default init system in most Linux distributions, like: Ubuntu, Fedora, Debian etc.

systemd is responsible for Initialising the system, managing services, and handling system states.
On systems which use systemd, a process called systemd is created with PID: 1, i.e. the first process created. Systemd replaces traditional init systems like SysVInit and Upstart

## Role of systemd:

- Initialize the system
- Manage services (start, stop, enable, disable)
- Handling dependencies: Ensuring services start in the correct order.
- Logging: systemd integrates with journald for centralized logging.

systemd process is the process that takes over right after the Linux Kernel boots, its job is to start and manage everything else.

To interact with systemd, we use the systemctl commands:
- Start a service: systemctl start <service_name>
- Stop a service: systemctl stop <service_name>
- Status of a service: systemctl status <service_name>

How is a service defined with systemd?
To define services we use unit files (ending in .service), a unit file specifies information such as:
- What command to run to start the service
- When should the command be run (ordering)
- What are the dependencies of the service
- If the process should be restarted on failure.

Note: Unit files in general represent any entity that can be managed by systemd. Services are one such entity.

How is the ordering determined?
Ordering is determined via dependencies (through “After”, and “Requuire”) and the desired target.

Dependencies:
After=network.target, means start the service after the networ.target unit
Requires=network.target, creates a hard dependency, i.e. the service can only be started if the network service is up.

Note, the semantics of After=, only specify ordering, i.e. the service should be started after another unit (X), it does not specify any dependency, so even if X could not be started, the current service can still be started.

Targets are used to group services:
Targets are liked named milestones or checkpoints during the boot, some of targets include:
- basic.target (basic system is up)
- network.target (network service is up)
- multi-user.target (system is ready for multiple users)

To specify the target, use the “WantedBy” attribute.

WantedBy=multi-user.target, means start the service when reaching multi-user.target

Attributes and their purposes:
- WantedBy: Grouping using targets
- After and Before: Ordering
- Requires: Hard Dependency (required)
- Wants: Soft Dependency (preferred, but not required)

Note: After and Before only affect the ordering, not the dependency.

To automatically start a service at boot, we make use of the “systemctl enable <service_name>” command.

Enabling and Disabling a service does not start or stop the service, they only control what happens at boot.
If a service is enabled, then a symlink pointing to the actual service unit file is created, so that when the system boots up, it can fetch this unit file and start the service.
Now, we can see where the WantedBy= attribute in the Install section fits in. When we specify WantedBy=multi-user.target, we say that this service should be started at boot as part of reaching the multi-user.target, more specifically, the symlink mentioned above is created in the directory multi-user.target.wants/, this directory contains symlinks to the unit files of all the services, which need to be started as part of reaching the multi-user.target milestone.
In simpler words, we make starting our service a prerequisite to the boot completion.

Sample Service unit file:

```text
[Unit]
Description=Resource Tuner Service
After=network.target

[Service]
User=root
Group=root
Restart=always
RemainAfterExit=yes
ExecStart=/usr/bin/resource_tuner --start
TimeoutStopSec=1

[Install]
WantedBy=multi-user.target
```

As can be seen, the service unit file contains 3 sections: Unit, Service and Install.

The Unit section contains metadata (like unit description) and dependencies.
Here we are saying start the service after the unit networ.target is up, though this is just an ordering constraint (After=), and not a dependency.

The Service section contains information about how the service runs.
- This includes ExecStart, which is the command to start the service.
- The restart policy: always or only in case of errors.
- User and Group, control permissions. Here we are saying run the service with root user (sudo) privileges (i.e. effective user id of the process will be root).

The install section controls service enablement. By enabling a service we mean that the service should be started automatically when the system boots. Hence, to enable a service, this Install section must be added.
The Install section has the WantedBy attribute which is used to specify the target (for example multi-user.target) as part of which the service is started.

When a unit file is created, it might not be picked up immediately by systemd, to force it to read and parse this unit file, use the systemctl daemon-reload command.

Additional Notes:
Targets are essentially named run levels, where run level is a construct used by older startup systems like SysVInit.
As the system boots it goes through all the targets sequentially.

## Difference b/w Requires= and After=
- Requires is used to establish a dependency (a hard dependency to be more specific). It states that the current unit can only be started, if the required unit has been correctly started and is up.
- If the required unit has not been started then it will be started by systemd (as stated in the Requires= attribute).
- If the required unit could not be started for any reasons, then the current unit won’t be started either. This is because for example, a service has a dependency on another service, and if that service is not up then the current service can’t boot or function properly. Such dependencies are expressed via Requires.
- As compared to Requires, the attribute After= has a very specific meaning. If two units are being started together (in other words the two units are part of the same transaction), then After=X, means start the service X first and then the current service. After and Before are required because systemd starts units parallelly to decrease boot time.
- If unit B has not been started yet, and we specify Requires=B, then the unit B will be satiated, however Requires does not guarantee that our unit will be started after B, since it is possible systemd starts them parallelly. It is this parallel startup, which can lead to problems if our service is expected to be started only after B has been started.
- Using a combination of Requires=X and After=X, implies that X is a dependency or prerequisite to the starting of the current service, i.e. if X could not be started then the current service can’t be started either. Cases:
  - If the target service X has already been started successfully, then Requires doesn’t need to do any additional thing.
  - If the target service X could not be started, then the current service won’t be started either (hard dependency)
  - If systemd notices that the target service X has not been already started, then it will start it. This is the case where After= comes into play, if we simply rely on Requires, then systemd will start the target service, however won’t guarantee any ordering. If we want to be sure that our service only starts after X has started, then the After attribute is needed.
  - The thing to keep in mind, is that systemd tries to minimize the boot time as much as possible, which means starting units in parallel wherever possible. After= attribute prevents this behaviour, and helps in controlling the boot process more explicitly.
