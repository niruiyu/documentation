Payload Interfaces
==================

The bootloader provides platform information to payload through
standard ACPI table, SMBIOS table, Device tree and a series of data
structures called the Hand Off Blocks (HOBs). If the information is
already defined in ACPI specification, SMBIOS specification or device
tree, the payload could parse them to get the required information.
For the platform information that is not defined in the standard
tables, the bootloader should build a HOB list to pass it to the
payload.

Open: Do we need a set of configuration data to config payload?

  We do not believe so.

Open: Do we need pass data from payload to bootloader to impact bootloader behavior in next boot?

  Keep it open now.

Open: will payload be run in S3 path?

  Suggest skipping payload.

.. _acpi_tables:

ACPI tables
-----------

ACPI table is required to boot modern operation system, especially to boot windows operating system.
The bootloader should provide a ACPI RSDP HOB. In the ACPI table least RSDT, FADT and MCFG should be available to the payload.
Payload could remove/add/modify the ACPI table passed from the bootloader if required.

The payload could parse the ACPI table to get some basic platform
information. For example, the Fixed ACPI Description Table (FADT)
defines various fixed hardware ACPI information to an ACPI compatible
OS, such as the base address for the following hardware registers
blocks: PM1a_CNT_BLK, PM_TMR_BLK, PM1a_EVT_BLK, GPE0_BLK,
PM1b_EVT_BLK, PM1b_CNT_BLK, PM2_CNT_BLK, and GPE1_BLK. The payload
could use them and other values (e.g. RESET_REG, RESET_VALUE) to make
the payload platform independent.

The other example is to get PCIE base address from ACPI memory mapped
configuration space access table definition, defined in the PCI
Firmware Specification.
`http://www.pcisig.com <http://www.pcisig.com/>`.

And another example is on the debug device info. The bootloader might
report debug device following up ACPI Debug Port Table 2 (DBG2). If a
fully 16550-compatible serial device is specified in the ACPI DBG2,
bootloader should provide a Serial Debug Information HOB in the HOB
list so that the payload could use same debug device with same
setting. If the ACPI DBG2 table could not be found, the payload
should use serial device provided by the Serial Debug Information HOB
as the default debug device.


.. hob_list:

HOB List
--------

The bootloader should build a HOB list and pass the HOB list header
to payload when passing control to payload. The HOB format is
described in the *Platform Initialization (PI) Specification - Volume
3: Shared Architectural Elements*.

There are two sections below describing the HOBs produced by the
bootloader and consumed by the payload for providing the system
architecturally information.

First section describes the HOBs defined in *Platform Initialization
Specification Volume 3: Shared Architectural elements*.

Second section defines the new HOBs.

Reusing Interfaces in Platform Initialization Specification
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

PHIT(Phase Handoff Info Table) HOB
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The bootloader should report the general state information through
the HOB following EFI_HOB_HANDOFF_INFO_TABLE format defined in
*Platform Initialization Specification Volume 3: Shared Architectural
elements*.

CPU HOB
^^^^^^^

The bootloader should report the processor information including address space
and I/O space capabilities to the payload through the HOB following
EFI_HOB_CPU format defined in *Platform Initialization Specification Volume 3:
Shared Architectural elements*.

Resource Descriptor HOB
^^^^^^^^^^^^^^^^^^^^^^^

The bootloader should report the system resources through the HOB
following EFI_HOB_RESOURCE_DESCRIPTOR format defined in *Platform
Initialization Specification Volume 3: Shared Architectural
elements*.

For example, any physical memory found in bootloader should be
reported using resource type EFI_RESOURCE_SYSTEM_MEMORY, and the
reserved memory used by bootloader should be reported using resource
type EFI_RESOURCE_MEMORY_RESERVED.

I/O and memory mapped I/O resource should also be reported using
resource type EFI_RESOURCE_IO and EFI_RESOURCE_MEMORY_MAPPED_IO.

Memory Allocation HOB
^^^^^^^^^^^^^^^^^^^^^

The bootloader should report the memory usages that exist outside the
HOB list through the HOB following EFI_HOB_MEMORY_ALLOCATION format defined
in *Platform Initialization Specification Volume 3: Shared Architectural
elements*.

Boot-Strap Processor (BSP) Stack Memory Allocation HOB
''''''''''''''''''''''''''''''''''''''''''''''''''''''

The bootloader should report the initial stack prepared for payload through
the HOB following EFI_HOB_MEMORY_ALLOCATION_STACK format defined in *Platform
Initialization Specification Volume 3: Shared Architectural elements*.

Memory Allocation Module HOB
''''''''''''''''''''''''''''

The bootloader should report the payload memory location and entry point
through the HOB following EFI_HOB_MEMORY_ALLOCATION_MODULE format defined
in *Platform Initialization Specification Volume 3: Shared Architectural
elements*.

Graphics information HOB
^^^^^^^^^^^^^^^^^^^^^^^^

If bootloader initializes the graphics device, the bootloader might
report graphics mode and framebuffer information through
EFI_PEI_GRAPHICS_INFO_HOB, and graphics hardware information
through EFI_PEI_GRAPHICS_DEVICE_INFO_HOB.

EFI_PEI_GRAPHICS_INFO_HOB and EFI_PEI_GRAPHICS_DEVICE_INFO_HOB provide the basic information
for the graphics display. These HOBs are described in the *PI Specification.*

Please refer Appendix 6.6 EFI_PEI_GRAPHICS_INFO_HOB and 6.7 EFI_PEI_GRAPHICS_DEVICE_INFO_HOB for the details.

New Interfaces
~~~~~~~~~~~~~~

Common Payload Header
^^^^^^^^^^^^^^^^^^^^^^^^^

All new interfaces are GUID type HOBs starting with ``EFI_HOB_GUID_TYPE`` defined in the PI Specification.

The HOB data starts with a common header defined as below::

  #pragma pack(1)
  
  typedef struct {
    UINT8                Revision;
    UINT8                Reserved;
    UINT16               Length;
  } UNIVERSAL_PAYLOAD_GENERIC_HEADER;

  #pragma pack()

``Revision``

It doesn't increase when new members are appended to the end of the interface.

It increases by one when existing members are renamed or re-interpreted for different purposes.

``Length``

The Length equals to the sizeof (UNIVERSAL_PAYLOAD_GENERIC_HEADER) + sizeof (<additional members>).

Consumers of the interfaces should only access those members that are covered by Length.

.. note::
  ``EFI_HOB_GUID_TYPE`` contains a Length field to tell the actual bytes the whole HOB data occupies.

  It also includes the optional padding bytes to make sure each HOB is multiple of 8 bytes in length.

  ``UNIVERSAL_PAYLOAD_GENERIC_HEADER.Length`` tells the exact length of the meaningful data excluding the padding bytes.
  So, it's always true that ``UNIVERSAL_PAYLOAD_GENERIC_HEADER.Length`` is less than or equal to the Length in ``EFI_HOB_GUID_TYPE``.

HOB data for different interfaces is defined in following sections.

ACPI Table
^^^^^^^^^^

The bootloader should pass ACPI table the payload. So that the payload could get the platform information from the ACPI table.

**GUID**

::

  gUniversalPayloadAcpiTableGuid = { 0x9f9a9506, 0x5597, 0x4515, { 0xba, 0xb6, 0x8b, 0xcd, 0xe7, 0x84, 0xba, 0x87 } }

**Structure**

::

  #pragma pack (1)

  typedef struct {
    UNIVERSAL_PAYLOAD_GENERIC_HEADER Header;
    EFI_PHYSICAL_ADDRESS             Rsdp;
  } UNIVERSAL_PAYLOAD_ACPI_TABLE;

  #pragma pack()

**Member Description**

``Header``

Header.Revision is 1.

Header.Length is 12.

``Rsdp``

Point to the ACPI RSDP table. The ACPI table need follow ACPI specification version 2.0 or above.

SMBIOS Table
^^^^^^^^^^^^

The bootloader might pass SMBIOS table to the payload. So that the payload could get the platform information from the table.

**GUID**

::

  gUniversalPayloadSmbios3TableGuid = { 0x92b7896c, 0x3362, 0x46ce, { 0x99, 0xb3, 0x4f, 0x5e, 0x3c, 0x34, 0xeb, 0x42 } }

  gUniversalPayloadSmbiosTableGuid = { 0x590a0d26, 0x06e5, 0x4d20, { 0x8a, 0x82, 0x59, 0xea, 0x1b, 0x34, 0x98, 0x2d } }

**Structure**

::

  #pragma pack (1)

  typedef struct {
    UNIVERSAL_PAYLOAD_GENERIC_HEADER Header;
    EFI_PHYSICAL_ADDRESS             SmBiosEntryPoint;
  } UNIVERSAL_PAYLOAD_SMBIOS_TABLE;

  #pragma pack()

**Member Description**

``Header``

Header.Revision is 1.

Header.Length is 12.

``SmBiosEntryPoint``

Points to the SMBIOS table in SMBIOS 3.0+ format if GUID is ``gUniversalPayloadSmbios3TableGuid``.

Points to the SMBIOS table in SMBIOS 2.x format if GUID is ``gUniversalPayloadSmbiosTableGuid``.

DEVICE TREE
^^^^^^^^^^^

The bootloader might pass Device Tree to the payload. So that the payload could get the platform information from the table.

**GUID**

::

  gUniversalPayloadDeviceTreeGuid = {0x6784b889, 0xb13c, 0x4c3b, {0xae, 0x4b, 0xf, 0xa, 0x2e, 0x32, 0xe, 0xa3}}

**Structure**

::

  #pragma pack (1)

  typedef struct {
    UNIVERSAL_PAYLOAD_GENERIC_HEADER Header;
    EFI_PHYSICAL_ADDRESS             DeviceTreeAddress;
  } UNIVERSAL_PAYLOAD_DEVICE_TREE;

  #pragma pack()

**Member Description**

``Header``

Header.Revision is 1.

Header.Length is 12.

``DeviceTreeAddress``

Point to the Device Tree entry point.

Serial Information
^^^^^^^^^^^^^^^^^^

If the debug device type and subtype are specified in DBG2, the
bootloader should 16550 compatible serial debug port information
to payload.

**Opens: Should we let bootloader provide debug callback** **for debug?**

**GUID**

::

  gUniversalPayloadSerialPortInfoGuid   = {0xaa7e190d, 0xbe21, 0x4409, {0x8e, 0x67, 0xa2, 0xcd, 0xf, 0x61, 0xe1, 0x70}}

**Structure**

::

  #pragma pack(1)

  typedef struct {
    UNIVERSAL_PAYLOAD_GENERIC_HEADER Header;
    BOOLEAN                          UseMmio;
    UINT8                            RegisterStride;
    UINT32                           BaudRate;
    EFI_PHYSICAL_ADDRESS             RegisterBase;
  } UNIVERSAL_PAYLOAD_SERIAL_PORT_INFO;

  #pragma pack()

**Member Description**

``Header``

Header.Revision is 1.

Header.Length is 18.

``UseMmio``

Indicates the 16550 serial port registers are in MMIO space, or in I/O space.

``RegisterStride``

Indicates the number of bytes between registers.

``BaudRate``

Baud rate for the 16550 compatible serial port.

It could be 921600, 460800, 230400, 115200, 57600, 38400, 19200,
9600, 7200, 4800, 3600, 2400, 2000, 1800, 1200, 600, 300, 150, 134,
110, 75, 50

Set to 0 to use the default baud rate 115200.

``RegisterBase``

Base address of 16550 serial port registers in MMIO or I/O space.

PCI Root Bridges
^^^^^^^^^^^^^^^^

The bootloader should pass information about PCI root bridges to the payload. The information tells the payload whether the PCI bus
enumeration has been performed by the bootloader, the bus, IO and MMIO ranges that are used or will be used by the PCI bus.

**GUID**

::

  gUniversalPayloadPciRootBridgeInfoGuid = { 0xec4ebacb, 0x2638, 0x416e, { 0xbe, 0x80, 0xe5, 0xfa, 0x4b, 0x51, 0x19, 0x01 }}

**Structure**

::

  #pragma pack(1)

  typedef struct {
    UNIVERSAL_PAYLOAD_GENERIC_HEADER  Header;
    BOOLEAN                           ResourceAssigned;
    UINT8                             Count;
    UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE RootBridge[0];
  } UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGES;

  typedef struct {
    UINT32                                     Segment;
    UINT64                                     Supports;
    UINT64                                     Attributes;
    BOOLEAN                                    DmaAbove4G;
    BOOLEAN                                    NoExtendedConfigSpace;
    UINT64                                     AllocationAttributes;
    UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE_APERTURE Bus;
    UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE_APERTURE Io;
    UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE_APERTURE Mem;
    UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE_APERTURE MemAbove4G;
    UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE_APERTURE PMem;
    UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE_APERTURE PMemAbove4G;
    UINT32                                     HID;
    UINT32                                     UID;
  } UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE;

  //
  // (Base > Limit) indicates an aperture is not available.
  //
  typedef struct {
    //
    // Base and Limit are the device address instead of host address when
    // Translation is not zero
    //
    UINT64 Base;
    UINT64 Limit;
    //
    // According to UEFI 2.7, Device Address = Host Address + Translation,
    // so Translation = Device Address - Host Address.
    // On platforms where Translation is not zero, the subtraction is probably to
    // be performed with UINT64 wrap-around semantics, for we may translate an
    // above-4G host address into a below-4G device address for legacy PCIe device
    // compatibility.
    //
    // NOTE: The alignment of Translation is required to be larger than any BAR
    // alignment in the same root bridge, so that the same alignment can be
    // applied to both device address and host address, which simplifies the
    // situation and makes the current resource allocation code in generic PCI
    // host bridge driver still work.
    //
    UINT64 Translation;
  } UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE_APERTURE;
  #pragma pack()

**Member Description**

``Header``

Header.Revision is 1.

Header.Length is 6 + ``Count`` * sizeof (UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE).

``ResourceAssigned``

Bus/IO/MMIO resources for all root bridges have been assigned when it's TRUE.

``Count``

Count of root bridges. Number of elements in ``RootBridge`` array.

``RootBridge[i].Segment``

Segment number of the root bridge.

``RootBridge[i].Supports``

Supported attributes. Refer to EFI_PCI_ATTRIBUTE_xxx used by GetAttributes() and SetAttributes() in EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL
defined in PI Specification.

``RootBridge[i].Attributes``

Initial attributes. Refer to EFI_PCI_ATTRIBUTE_xxx used by GetAttributes() and SetAttributes() in EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL
defined in PI Specification.

``RootBridge[i].DmaAbove4G``

Root bridge supports DMA above 4GB memory when it's TRUE.

``RootBridge[i].NoExtendedConfigSpace``

Root bridge supports 256-byte configuration space only when it's TRUE.
Root bridge supports 4K-byte configuration space when it's FALSE.

``RootBridge[i].AllocationAttributes``

Allocation attributes. Refer to EFI_PCI_HOST_BRIDGE_COMBINE_MEM_PMEM and EFI_PCI_HOST_BRIDGE_MEM64_DECODE used by GetAllocAttributes()
in EFI_PCI_HOST_BRIDGE_RESOURCE_ALLOCATION_PROTOCOL defined in PI Specification.

``RootBridge[i].Bus``

Bus aperture for the root bridge.

``RootBridge[i].Io``

IO aperture for the root bridge.

``RootBridge[i].Mem``

MMIO aperture below 4GB for the root bridge.

``RootBridge[i].MemAbove4G``

MMIO aperture above 4GB for the root bridge.

``RootBridge[i].PMem``

Prefetchable MMIO aperture below 4GB for the root bridge.

``RootBridge[i].PMemAbove4G``

Prefetchable MMIO aperture above 4GB for the root bridge.

``RootBridge[i].HID``

PnP hardware ID of the root bridge. This value must match the corresponding _HID in the ACPI name space.

``RootBridge[i].UID``

Unique ID that is required by ACPI if two devices have the same _HID. This value must also match the corresponding _UID/_HID pair in the ACPI name space.

Optional Interfaces
~~~~~~~~~~~~~~~~~~~

Some more HOBs could be built by bootloaders for advanced features. e.g.:

  Support FVs (also other format) from bootloader to payload

  Add debug log as HOB to payload

**Opens**: Does the bootloader need report IO info to payload?

      Better let the bootloader to report it,

**Opens**: does the HOB List need a checksum?

      It looks not too much value. Keep it open if we really need it.

**Opens**: For some information it is already in ACPI table, should bootloader build HOB for same info?

      Payload could have a check to ACPI table to get basic info they need.

