---
title: "CA1063: Implement IDisposable correctly | Microsoft Docs"
ms.custom: ""
ms.date: "11/04/2016"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "vs-devops-test"
ms.tgt_pltfrm: ""
ms.topic: "article"
f1_keywords: 
  - "ImplementIDisposableCorrectly"
  - "CA1063"
helpviewer_keywords: 
  - "CA1063"
  - "ImplementIDisposableCorrectly"
ms.assetid: 12afb1ea-3a17-4a3f-a1f0-fcdb853e2359
caps.latest.revision: 17
author: "stevehoag"
ms.author: "shoag"
manager: "wpickett"
translation.priority.ht: 
  - "de-de"
  - "es-es"
  - "fr-fr"
  - "it-it"
  - "ja-jp"
  - "ko-kr"
  - "ru-ru"
  - "zh-cn"
  - "zh-tw"
translation.priority.mt: 
  - "cs-cz"
  - "pl-pl"
  - "pt-br"
  - "tr-tr"
---
# CA1063: Implement IDisposable correctly
|||  
|-|-|  
|TypeName|ImplementIDisposableCorrectly|  
|CheckId|CA1063|  
|Category|Microsoft.Design|  
|Breaking Change|Non-breaking|  
  
## Cause  
 `IDisposable` is not implemented correctly. Some reasons for this problem are listed here:  
  
-   IDisposable is re-implemented in the class.  
  
-   Finalize is re-overridden.  
  
-   Dispose is overridden.  
  
-   Dispose() is not public, sealed, or named Dispose.  
  
-   Dispose(bool) is not protected, virtual, or unsealed.  
  
-   In unsealed types, Dispose() must call Dispose(true).  
  
-   For unsealed types, the Finalize implementation does not call either or both Dispose(bool) or the case class finalizer.  
  
 Violation of any one of these patterns will trigger this warning.  
  
 Every unsealed root IDisposable type must provide its own protected virtual void Dispose(bool) method. Dispose() should call Dipose(true) and Finalize should call Dispose(false). If you are creating an unsealed root IDisposable type, you must define Dispose(bool) and call it. For more information, see [Cleaning Up Unmanaged Resources](http://msdn.microsoft.com/Library/a17b0066-71c2-4ba4-9822-8e19332fc213) in the [Framework Design Guidelines](http://msdn.microsoft.com/Library/5fbcaf4f-ea2a-4d20-b0d6-e61dee202b4b) section of the .NET Framework documentation.  
  
## Rule Description  
 All IDisposable types should implement the Dispose pattern correctly.  
  
## How to Fix Violations  
 Examine your code and determine which of the following resolutions will fix this violation.  
  
-   Remove IDisposable from the list of interfaces that are implemented by {0} and override the base class Dispose implementation instead.  
  
-   Remove the finalizer from type {0}, override Dispose(bool disposing), and put the finalization logic in the code path where 'disposing' is false.  
  
-   Remove {0}, override Dispose(bool disposing), and put the dispose logic in the code path where 'disposing' is true.  
  
-   Ensure that {0} is declared as public and sealed.  
  
-   Rename {0} to 'Dispose' and make sure that it is declared as public and sealed.  
  
-   Make sure that {0} is declared as protected, virtual, and unsealed.  
  
-   Modify {0} so that it calls Dispose(true), then calls GC.SuppressFinalize on the current object instance ('this' or 'Me' in [!INCLUDE[vbprvb](../code-quality/includes/vbprvb_md.md)]), and then returns.  
  
-   Modify {0} so that it calls Dispose(false) and then returns.  
  
-   If you are writing an unsealed root IDisposable class, make sure that the implementation of IDisposable follows the pattern that is described earlier in this section.  
  
## When to Suppress Warnings  
 Do not suppress a warning from this rule.  
  
## Pseudo-code Example  
 The following pseudo-code provides a general example of how Dispose(bool) should be implemented in a class that uses managed and native resources.  
  
```  
public class Resource : IDisposable   
{  
    private IntPtr nativeResource = Marshal.AllocHGlobal(100);  
    private AnotherResource managedResource = new AnotherResource();  
  
// Dispose() calls Dispose(true)  
    public void Dispose()  
    {  
        Dispose(true);  
        GC.SuppressFinalize(this);  
    }  
    // NOTE: Leave out the finalizer altogether if this class doesn't   
    // own unmanaged resources itself, but leave the other methods  
    // exactly as they are.   
    ~Resource()   
    {  
        // Finalizer calls Dispose(false)  
        Dispose(false);  
    }  
    // The bulk of the clean-up code is implemented in Dispose(bool)  
    protected virtual void Dispose(bool disposing)  
    {  
        if (disposing)   
        {  
            // free managed resources  
            if (managedResource != null)  
            {  
                managedResource.Dispose();  
                managedResource = null;  
            }  
        }  
        // free native resources if there are any.  
        if (nativeResource != IntPtr.Zero)   
        {  
            Marshal.FreeHGlobal(nativeResource);  
            nativeResource = IntPtr.Zero;  
        }  
    }  
}  
```