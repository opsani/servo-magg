# servo-magg
Measure Driver aggregator

The measure aggregator can be used to combine several measure drivers compatible with the Optune `servo` agent and present them all as a single driver.

## Operation

The aggregator behaves like a normal `measure` driver and will be recognized as such by the `servo` agent. It runs all drivers located in a directory `./measure.d/`. Any executable file in that directory is considered to be a driver. All drivers are run simultaneously, so naming and order are not important. When running a sub-driver, the aggregator runs it with a current directory that is the same as the one for the aggregator. This allows sub-drivers to access their config file as `./config.yaml`, thus being unaware that they are run as part of an aggregator. The aggregator fails unless there is at least one sub-driver.

Progress is reported as the minimum progress of all sub-drivers.


## Setup

To use the aggregator:

- place the executable and the Python libraries into a single directory, e.g., `servo`
- in the same directory, create a sub-directory `measure.d` and place all drivers that are to be aggregated into it. If those drivers need supporting files (Python modules, data files, etc.), also place them in `measure.d`.
- the configuration file should be placed in the root directory where the aggregator is (not in `measure.d`). This root directory should be the current directory when the aggregator is run (this is also where one would place the `servo` agent, just as one would when configuring it with a single measure driver).

The resulting directory structure should look like this:

/servo/             # this will be the current directory when ‘measure’ is ran
      /config.yaml  # configuration for all drivers
      /measure      # the aggregator executable
      /measure.py   # common library for measure drivers (optional)
      /adjust       # adjust driver
      /adjust.py    # common library for adjust drivers (optional)
      /servo        # optune servo agent
      /measure.d/   # drivers directory (see NOTE)
            /measure.py   # common library for measure drivers (optional)
            /driver1  # executable
            /driver2  # executable
            /datafile # not executable

NOTE:the only files in `measure.d` that have the executable flag should be one or more `measure` drivers. If the driver requires any supporting executables of its own, they should be installed elsewhere (OK to place them in a sub-directory of `measure.d`)

After copying all files, the setup can be tested by running:

	./measure --info


## Configuration

The aggregator does not require any configuration, but can optionally take the following config parameters, specified in `config.yaml` (same file as all other measure and adjust drivers) under a `magg` key:
  * `grace_period` - specify how long (in seconds) to wait for sub-drivers to complete a measurement. This time is added to the sum of warmup + duration + past (as specified in the control section of the input). If there is a per-driver overrides in the control section, the longest value for warmup + duration + past of any driver is used. If configured, this will terminate all drivers even if they do not support cancel. If set to `0`, no timeout is enforced. Default: `0`
  * `force_cancel` - if set to `True`, the aggregator will terminate all sub-drivers immediately on error. If set to `False`, the aggregator will check if all sub-drivers support cancelling - if yes it will terminate them immediately, otherwise it will wait indefinitely for drivers to exit before it returns the error and exits. Default: `False`
  * `cleanup_grace_period` - how long (in seconds) to wait when stopping running drivers (this will happen if any driver fails or if a timeout occurs). As part of the cleanup, the aggregator sends SIGUSR1 to all running drivers, then waits up to `cleanup_grace_period` seconds and then sends SIGKILL to all drivers that are still running. Default: `300`




