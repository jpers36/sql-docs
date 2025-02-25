---
title: "CursorLocationEnum"
description: "CursorLocationEnum"
author: rothja
ms.author: jroth
ms.date: "01/19/2017"
ms.prod: sql
ms.technology: ado
ms.topic: reference
f1_keywords:
  - "CursorLocationEnum"
helpviewer_keywords:
  - "CursorLocationEnum enumeration [ADO]"
apitype: "COM"
---
# CursorLocationEnum
Specifies the location of the cursor service.  
  
|Constant|Value|Description|  
|--------------|-----------|-----------------|  
|**adUseClient**|3|Uses client-side cursors supplied by a local cursor library. Local cursor services often will allow many features that driver-supplied cursors may not, so using this setting may provide an advantage with respect to features that will be enabled. For backward compatibility, the synonym **adUseClientBatch** is also supported.|  
|**adUseNone**|1|Does not use cursor services. (This constant is obsolete and appears solely for the sake of backward compatibility.)|  
|**adUseServer**|2|Default. Uses cursors supplied by the data provider or driver. These cursors are sometimes very flexible and allow for additional sensitivity to changes others make to the data source. However, some features of the [The Microsoft Cursor Service for OLE DB](../../guide/data/the-microsoft-cursor-service-for-ole-db.md), such as disassociated<br /><br /> [Recordset](./recordset-object-ado.md) objects, cannot be simulated with server-side cursors and these features will be unavailable with this setting.|  
  
## ADO/WFC Equivalent  
 Package: **com.ms.wfc.data**  
  
|Constant|  
|--------------|  
|AdoEnums.CursorLocation.CLIENT|  
|AdoEnums.CursorLocation.NONE|  
|AdoEnums.CursorLocation.SERVER|  
  
## Applies To  
 [CursorLocation Property (ADO)](./cursorlocation-property-ado.md)