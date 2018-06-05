---
title: GetPortAttributesByWWN\_IN structure
description: The GetPortAttributesByWWN\_IN structure is used by a WMI client to deliver input parameter data to the GetPortAttributesByWWN WMI method.
ms.assetid: 2b189ece-6c49-42e2-8ef2-b3db516fc844
keywords:
- GetPortAttributesByWWN_IN structure Storage Devices
- PGetPortAttributesByWWN_IN structure pointer Storage Devices
topic_type:
- apiref
api_name:
- GetPortAttributesByWWN_IN
api_location:
- hbapiwmi.h
api_type:
- HeaderDef
ms.technology: desktop
ms.prod: windows
ms.author: windowssdkdev
ms.topic: structure
ms.date: 05/31/2018
---

# GetPortAttributesByWWN\_IN structure

The GetPortAttributesByWWN\_IN structure is used by a WMI client to deliver input parameter data to the [**GetPortAttributesByWWN**](https://msdn.microsoft.com/library/windows/hardware/ff554965) WMI method.

## Syntax


```C++
typedef struct _GetPortAttributesByWWN_IN {
  UCHAR wwn[8];
} GetPortAttributesByWWN_IN, *PGetPortAttributesByWWN_IN;
```



## Members

<dl> <dt>

**wwn**
</dt> <dd>

Contains a worldwide name that identifies a port of type FC\_Port.

</dd> </dl>

## Remarks

The WMI tool suite generates a declaration of the GetPortAttributesByWWN\_IN structure in *Hbapiwmi.h* when it compiles the [MSFC\_HBAAdapterMethods WMI Class](https://msdn.microsoft.com/library/windows/hardware/ff562506).

For a definition of FC\_Port and a discussion of worldwide names, see the T11 committee's specification for *Fibre Channel HBA API* (FC-HBA).

## Requirements



|                   |                                                                                                            |
|-------------------|------------------------------------------------------------------------------------------------------------|
| Header<br/> | <dl> <dt>Hbapiwmi.h (include Hbapiwmi.h)</dt> </dl> |



## See also

<dl> <dt>

[**GetPortAttributesByWWN**](https://msdn.microsoft.com/library/windows/hardware/ff554965)
</dt> </dl>

 

 

[Send comments about this topic to Microsoft](mailto:wsddocfb@microsoft.com?subject=Documentation%20feedback%20%5Bstorage\storage%5D:%20GetPortAttributesByWWN_IN%20structure%20%20RELEASE:%20%283/29/2018%29&body=%0A%0APRIVACY%20STATEMENT%0A%0AWe%20use%20your%20feedback%20to%20improve%20the%20documentation.%20We%20don't%20use%20your%20email%20address%20for%20any%20other%20purpose,%20and%20we'll%20remove%20your%20email%20address%20from%20our%20system%20after%20the%20issue%20that%20you're%20reporting%20is%20fixed.%20While%20we're%20working%20to%20fix%20this%20issue,%20we%20might%20send%20you%20an%20email%20message%20to%20ask%20for%20more%20info.%20Later,%20we%20might%20also%20send%20you%20an%20email%20message%20to%20let%20you%20know%20that%20we've%20addressed%20your%20feedback.%0A%0AFor%20more%20info%20about%20Microsoft's%20privacy%20policy,%20see%20http://privacy.microsoft.com/default.aspx. "Send comments about this topic to Microsoft")





