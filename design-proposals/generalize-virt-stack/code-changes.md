# CRD structure

## Example scenario with qemu

```yaml
hypervisors:
  qemu-kvm:
    virtLauncherOverheads:
      virtLauncherMonitorOverhead: 25Mi
      virtLauncherOverhead: 100Mi
      virtlogdOverhead: 20Mi
      hypervisorDaemonOverhead: 35Mi
      hypervisorOverhead: 30Mi
    hypervisorDevice: devices.kubevirt.io/kvm
    shouldRunPrivileged: false
    vcpuRegex: ^CPU (\d+)/KVM\n$
    runtimeUser:
      root:
        pidDir: /run/libvirt/qemu
        libvirtUri: qemu+unix:///session?socket=/var/run/libvirt/virtqemud-sock
      nonRoot:
        pidDir: /run/libvirt/qemu/run
        libvirtUri: qemu:///system
    hypervisorCommandPrefixes:
    - qemu-system
    - qemu-kvm
    libvirtDaemon:
      libvirtDaemonExecutable: /usr/sbin/virtqemud
      libvirtDaemonConfig: /var/run/libvirt/virtqemud.conf
    diskDriver: qemu  
    cloudInitFileType: iso



```

# Interface draft

## Interfaces

There are two interfaces that encapsulate the specific needs of different virtualization stacks.

### LibvirtWrapper interface

This interface encapsulates the logic for setting up and launching the hypervisor daemon (e.g., virtqemud or libvirtd) and exposes the following functions:

- SetupLibvirt: Setup libvirt for hosting the virtual machine. This function is called during the startup of the virt-launcher.
- GetHypervisorCommandPrefix: Return a list of potential prefixes of the specific hypervisor's process, e.g., qemu-system or cloud-hypervisor
- StartHypervisorDaemon: Start the libvirt daemon, either in modular mode or monolithic mode
- StartVirtlog: Start the virtlogd daemon, which is used to capture logs from the hypervisor
- GetLibvirtUriAndUser: Returns true if the libvirt process(es) should be run as root user
- GetPidDir: Return the directory where libvirt stores the PID files of the hypervisor processes

### Hypervisor interface

This interface provides 

## Code usage of Hypervisor interface

- pkg/virt-controller/services/renderresources.go: GetMemoryOverhead

  - GetVirtLauncherMonitorOverhead

- pkg/virt-controller/services/renderresources.go: GetRequiredResources

  This function sets the requirement of host device resource `devices.kubevirt.io/kvm` on the virt-launcher pod.

  - GetHypervisorDevice

- pkg/virt-controller/services/template.go: newContainerSpecRenderer

  Hypervisor interface is used to determine whether the virt-launcher container should be run in Privileged mode.

  - ShouldRunPrivileged

- pkg/virt-handler/realtime.go: configureVCPUScheduler

  Hypervisor interface is used to get the vCPU regex used by the hypervisor.

  - GetVcpuRegex

- pkg/virt-handler/vm.go: handleTargetMigrationProxy

  - Used to get the Libvirt socket file on the destination pod

- pkg/virt-handler/vm.go: IsoGuestVolumePath

  - Used to check whether the hypervisor supports ISO files, and accordingly return the path of the cloudInit file.

    SupportsIso

- pkg/virt-handler/vm.go: vmUpdateHelperMigrationTarget, vmUpdateHelperDefault

  Used to get the name of the hypervisor device (e.g., /dev/kvm) so that virt-handler can claim ownership of that device for the VM.

  GetHypervisorDevice

- pkg/virt-handler/vm.go: affinePitThread

  Used to get the vCPU regex.

  GetVcpuRegex

- pkg/virt-launcher/virtwrap/manager.go: generateConverterContext



- pkg/virt-launcher/virtwrap/manager.go: generateSomeCloudInitISO

  SupportsIso

- pkg/virt-launcher/virtwrap/converter/converter.go: Convert_v1_VirtualMachineInstance_To_api_Domain

  GetHypervisorDevice

  SupportsMemoryBallooning


***Hypervisor is made a part of the ConverterContext***

## Code usage of LibvirtWrapper interface

- SetupLibvirt()

  - Called from cmd/virt-launcher/virt-launcher.go: main()

- GetHypervisorCommandPrefix

  - Called from cmd/virt-launcher-monitor/virt-launcher-monitor.go: RunAndMonitor()

- StartHypervisorDaemon()

  - Called from cmd/virt-launcher/virt-launcher.go: main()

- StartVirtlog

  - Called from cmd/virt-launcher/virt-launcher.go: main()

- GetLibvirtUriAndUser

  - Called from cmd/virt-launcher/virt-launcher.go: createLibvirtConnection()

- GetPidDir

  - Called from cmd/virt-launcher/virt-launcher.go: main()