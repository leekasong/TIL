# WDM 2


### 기본 코드
```
#include "Driver.h"

VOID
MyUnload(
	_In_ struct _DRIVER_OBJECT *DriverObject
)
{
	UNREFERENCED_PARAMETER(DriverObject);
	DbgPrintEx(DPFLTR_IHVDRIVER_ID, 0, "WDM Driver Unloading...\n");
}



NTSTATUS
MyAddDevice(
	_In_ PDRIVER_OBJECT DriverObject,
	_In_ PDEVICE_OBJECT PhysicalDeviceObject
) {
	NTSTATUS nStatus = STATUS_UNSUCCESSFUL;
	PMYDEVICE_EXTENSION pMyDeviceExtension = NULL;
	PDEVICE_OBJECT MyDeviceObject;
	//디바이스 오브젝트 생성 -> MyDeviceObject에 담는다.
	nStatus = IoCreateDevice(
		DriverObject,
		//디바이스 익스텐션(디바이스 오브젝트가 사용할 정보 저장 공간)
		sizeof(MYDEVICE_EXTENSION),
		NULL,
		FILE_DEVICE_UNKNOWN,
		FILE_AUTOGENERATED_DEVICE_NAME,
		FALSE,
		&MyDeviceObject
	);
	if (!NT_SUCCESS(nStatus)) goto exit;


	pMyDeviceExtension = MyDeviceObject->DeviceExtension;
	RtlZeroMemory(pMyDeviceExtension, sizeof(MYDEVICE_EXTENSION));
	//여기서는 디바이스 익스텐션에 내가 어태치되기 직전에 최상위 오브젝트였던 놈을 저장
	//그 값은 어태치 함수의 반환값으로 얻는다.
	pMyDeviceExtension->NextDeviceObject =
		IoAttachDeviceToDeviceStack(
			MyDeviceObject,
			PhysicalDeviceObject
		);

	//나의 디바이스 오브젝트가 초기화 과정을 마쳤다고 설정
	MyDeviceObject->Flags &= DO_DEVICE_INITIALIZING;
exit:
	DbgPrintEx(DPFLTR_IHVDRIVER_ID, 0, "WDM MyAddDevice -- %8X\n", nStatus);
	return nStatus;
}

//플러그앤플레이 IRP 처리를 위한 콜백함수
NTSTATUS
MyPnPIRPDispatch(
	_In_ PDEVICE_OBJECT DeviceObject,
	_Inout_ PIRP Irp
) {
	NTSTATUS ntStatus = STATUS_NOT_SUPPORTED;
	//내 디바이스 오브젝트의 디바이스 익스텐션을 얻는다.
	PMYDEVICE_EXTENSION pDeviceExtension = (PMYDEVICE_EXTENSION)DeviceObject->DeviceExtension;
	//익스텐션에 있는 다음 디바이스 오브젝트의 포인터를 얻는다.
	PDEVICE_OBJECT pNextDevObj = pDeviceExtension->NextDeviceObject;
	//현재 디바이스 오브젝트 스택 로케이션을 얻는다.
	PIO_STACK_LOCATION pStack = IoGetCurrentIrpStackLocation(Irp);
	switch (pStack->MinorFunction)
	{
		//스택에 기록한 값이 IRP_MN_REMOVE_DEVICE
		//이 값은 디바이스 오브젝트 제거 명령이다.
	case IRP_MN_REMOVE_DEVICE:
		//내 아래에 있는 디바이스 오브젝트와 디태치
		IoDetachDevice(pNextDevObj);
		//그리고 내 디바이스 오브젝트를 삭제한다.
		IoDeleteDevice(DeviceObject);
		break;
	default:
		break;
	}

	//디바이스 스택 인덱스를 증가. 즉 팝
	IoSkipCurrentIrpStackLocation(Irp);
	//다음 디바이스 오브젝트에게 IRP을 넘겨준다.
	ntStatus = IoCallDriver(pNextDevObj, Irp);
	return ntStatus;
}

NTSTATUS
DriverEntry(
	IN PDRIVER_OBJECT DriverObject,
	IN PUNICODE_STRING RegistryPath
)
{
	UNREFERENCED_PARAMETER(RegistryPath);

	DbgPrintEx(DPFLTR_IHVDRIVER_ID, 0, "WDM Driver loading...\n");
	DriverObject->DriverUnload = MyUnload;
	DriverObject->DriverExtension->AddDevice = MyAddDevice;
	DriverObject->MajorFunction[IRP_MJ_PNP] = MyPnPIRPDispatch;

	return STATUS_SUCCESS;
}
```

### 설치정보
* inf 파일에 작성한다.
* 섹션 별로 정의된다.

#### Version
* 설치파일에 대한 버전정보  

#### ClassInstall32
* 새로운 클래스를 추가할 때 필요한 정보를 담는다.
* 반드시 정의해야 함.  

#### DestinationDirs
* 드라이버를 복사할 경로를 명시  

#### SourceDisksNames
* 드라이버를 가져올 원본 경로의 디스크 이름  

#### SourceDisksFiles
* SourceDisksNames 디스크 내의 파일 정보  

#### Manufacturer
* 드라이버 만든 사람 및 드라이버 제품 정보  

#### Strings
* 설치파일내의 사용할 문자열 정보  

### inf 파일 예시
```
;
; ExamDriver2.inf
;

[Version]
Signature="$WINDOWS NT$"
Class=%ClassName%
ClassGuid={29044855-770B-4232-8BF0-7F66F8A71C5E}
Provider=%ManufacturerName%
DriverVer=
CatalogFile=ExamDriver2.cat

[DestinationDirs]
DefaultDestDir = 12

[ClassInstall32]
Addreg=AddHwClass

[AddHWClass]
HKR,,,,%ClassName%

[SourceDisksNames]
1 = %DiskName%,,,""

[SourceDisksFiles]
ExamDriver2.SYS = 1

[Manufacturer]
%ManufacturerName%=Standard,NT$ARCH$

[Standard.NT$ARCH$]
%DeviceDesc%= ExamDriver2, LEEKASONG_SIMPLEDEVICE ; TODO: Driver id

[ExamDriver2]
CopyFiles=@ExamDriver2.SYS

[ExamDriver2.Services]
AddService = ExamDriver2,2,Exam_Service_Inst

[Exam_Service_Inst]
ServiceType = 1
StartType = 3
ErrorControl = 1
ServiceBinary = %12%\ExamDriver2.SYS

[Strings]
ManufacturerName="LEEKASONG" ;TODO: Replace with your manufacturer name
ClassName="ExamDriver2"
DiskName="ExamDriver2 Source Disk"
DeviceDesc="ExamDriver2 Device"
```
* ClassInstall32 꼭 작성
* [Standard.NT$ARCH$] 섹션에서 드라이버 id를 작성한다.
* 서비스 형태로 동작하기 때문에 서비스 섹션을 두고, 필요한 내용을 작성