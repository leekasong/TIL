# 피씨 자원을 얻는 클래스입니다.

### 헤더

```
#pragma once
class CResourceStatus
{
protected:
	//QUERY
	HCOUNTER m_hQuery;
	//user defined value.
	DWORD_PTR m_userValue;
	//CPU
	HCOUNTER m_hCPUTotalStatus;
	//NETWORK
	HCOUNTER *m_pNetworkStatusList;
	DWORD m_dwNetworkCounterNumber;
protected:
	BOOL Init();
	void Terminate();
public:
	//쿼리를 날려서 데이터를 갱신.
	inline void QueryData(){PdhCollectQueryData(m_hQuery);}
	BOOL GetCPU_Usage(DWORD *pCPUData);
	BOOL GetNetwork_Usage(DWORD **pNetworkData);
	inline DWORD GetNetworkCounterNumber() { return m_dwNetworkCounterNumber; }
	BOOL GetMemory_Usage(DWORD *pTotalMemory, DWORD *pUsedMemory);
	//디스크 정보외에도 더 추가될 예정
	BOOL ScanHW(CStringList *pHWList);
	BOOL ScanSW(CStringList *pSWList);
public:
	CResourceStatus();
	~CResourceStatus();
};
```

### CPP
```
#include "stdafx.h"
#include "ResourceStatus.h"


CResourceStatus::CResourceStatus()
	:m_hQuery(NULL)
	, m_hCPUTotalStatus(NULL)
	, m_pNetworkStatusList(NULL)
{

	m_userValue = 25001;
	m_dwNetworkCounterNumber = 0;
	if (!Init()) {
		AfxMessageBox(_T("실패"));
	}
}


CResourceStatus::~CResourceStatus()
{
	Terminate();



}

BOOL CResourceStatus::Init() {

	//데이터를 받아오기 위해 쿼리 핸들을 구하는 함수
	//NULL을 주면 실시간으로 데이터를 받는다.
	PDH_STATUS res = PdhOpenQuery(NULL, m_userValue, &m_hQuery);
	if (res != ERROR_SUCCESS) return FALSE;

	/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
	//CPU 카운터 등록
	res = PdhAddCounter(m_hQuery, _T("\\Processor(_Total)\\% Processor Time"), m_userValue, &m_hCPUTotalStatus);
	if (res != ERROR_SUCCESS) return FALSE;

	/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
	//네트워크
	DWORD dwCounterListLength = 0, dwInstanceLength = 0;
	//버퍼 크기 얻기
	//세번째인자는 네트워크 자원에 대한 오브젝트 이름
	PdhEnumObjectItems(NULL, NULL, _T("Network Interface"), NULL, &dwCounterListLength,
		NULL, &dwInstanceLength, PERF_DETAIL_WIZARD, 0);

	TCHAR *pCounterNameList = new TCHAR[dwCounterListLength]{ 0 };
	TCHAR *pInstanceNameList = new TCHAR[dwInstanceLength]{ 0 };

	//네트워크 자원 인스턴스 이름과 카운터 이름 얻기
	res = PdhEnumObjectItems(NULL, NULL, _T("Network Interface"), pCounterNameList, &dwCounterListLength
		, pInstanceNameList, &dwInstanceLength, PERF_DETAIL_WIZARD, 0);

	if (res != ERROR_SUCCESS) return FALSE;

	TCHAR *pTemp = pInstanceNameList;
	//발견된 네트워크 자원 개수 얻기
	while (*pTemp) {
		m_dwNetworkCounterNumber++;
		pTemp += (1 + _tcslen(pTemp));
	}

	TCHAR counterPathBuf[MAX_PATH] = { 0 };
	DWORD dwBufferSize = sizeof(counterPathBuf) / sizeof(TCHAR);
	m_pNetworkStatusList = new HCOUNTER[m_dwNetworkCounterNumber]{ 0 };
	pTemp = pInstanceNameList;

	//해당 네트워크 자원에 대한 카운터 핸들 얻기
	for (int i = 0; i < m_dwNetworkCounterNumber; i++) {
		::ZeroMemory(counterPathBuf, sizeof(counterPathBuf));
		dwBufferSize = sizeof(counterPathBuf) / sizeof(TCHAR);

		//카운터 경로 구하기 위한 구조체 설정
		PDH_COUNTER_PATH_ELEMENTS pcpe = { 0 };
		pcpe.dwInstanceIndex = i;
		pcpe.szCounterName = _T("Bytes Total/sec");
		pcpe.szInstanceName = pTemp;
		pcpe.szObjectName = _T("Network Interface");
		PdhMakeCounterPath(&pcpe, counterPathBuf, &dwBufferSize, 0);
		//카운터 핸들 얻기
		res = PdhAddCounter(m_hQuery, counterPathBuf, m_userValue, &m_pNetworkStatusList[i]);
		if (res != ERROR_SUCCESS) return FALSE;
		pTemp += (1 + _tcslen(pTemp));
	}


	delete[] pCounterNameList;
	delete[] pInstanceNameList;
	return TRUE;

}

void CResourceStatus::Terminate() {
	PdhRemoveCounter(m_hCPUTotalStatus);
	for (int i = 0; i < m_dwNetworkCounterNumber; i++) {
		PdhRemoveCounter(m_pNetworkStatusList[i]);
	}
	delete[] m_pNetworkStatusList;
}

BOOL CResourceStatus::GetCPU_Usage(DWORD *pCPUData) {
	PDH_FMT_COUNTERVALUE pfc = { 0 };

	PDH_STATUS ps = PdhGetFormattedCounterValue(
		m_hCPUTotalStatus,
		PDH_FMT_LONG,
		NULL,
		&pfc
	);
	if (ps != ERROR_SUCCESS) return FALSE;
	*pCPUData = pfc.longValue;
	return TRUE;
}

BOOL CResourceStatus::GetNetwork_Usage(DWORD **pNetworkData) {
	PDH_FMT_COUNTERVALUE pfc = { 0 };

	//갱신된 데이터를 변수에 저장.
	for (int i = 0; i < m_dwNetworkCounterNumber; i++) {
		PDH_STATUS ps = PdhGetFormattedCounterValue(
			m_pNetworkStatusList[i],
			PDH_FMT_LONG,
			NULL,
			&pfc
		);
		if (ps != ERROR_SUCCESS) return FALSE;
		*(*(pNetworkData)+i) = pfc.longValue;
	}
	return TRUE;
}

BOOL CResourceStatus::GetMemory_Usage(DWORD *pTotalMemory, DWORD *pUsedMemory) {
	MEMORYSTATUSEX mse = { 0 };
	mse.dwLength = sizeof(mse);
	::GlobalMemoryStatusEx(&mse);

	//메모리 총량
	*pTotalMemory = mse.ullTotalPhys / (1024 * 1024);
	//메모리 사용량
	*pUsedMemory = (mse.ullTotalPhys - mse.ullAvailPhys) / (1024 * 1024);
	return TRUE;
}

BOOL CResourceStatus::ScanHW(CStringList *pHWList) {

	///////////////////////////////////////////////////////////////////////////
	//디스크 용량 파악
	//A~Z가 비트단위로 저장
	DWORD bitmask = GetLogicalDrives();
	DWORD bit = 1;
	for (char c = 'A'; c <= 'Z'; c++) {
		TCHAR rootPath[5] = { 0 };
		if (bit & bitmask) {
			ULARGE_INTEGER totalDiskSpace = { 0 }, availableDiskSpace = { 0 };
			_stprintf(rootPath, _T("%C:"), c);
			UINT type = GetDriveType(rootPath);
			if (type == DRIVE_FIXED) {
				//드라이브 용량 구하기
				GetDiskFreeSpaceEx(rootPath, NULL, &totalDiskSpace, &availableDiskSpace);
				CString diskData = _T("");
				diskData.Format(
					_T("%C Disk : %I64d / %I64d (GB)\r\n")
					, c
					, (totalDiskSpace.QuadPart - availableDiskSpace.QuadPart) / (1024 * 1024 * 1024)
					, (totalDiskSpace.QuadPart) / (1024 * 1024 * 1024)
				);

				pHWList->AddTail(diskData);
			}
		}
		bit *= 2;
	}

	///////////////////////////////////////////////////////////////////////////
	//프로세서 이름과 개수

	SYSTEM_INFO si = { 0 };
	GetSystemInfo(&si);
	DWORD dwProcessorNumber = si.dwNumberOfProcessors;
	TCHAR basicPath[] = _T("Hardware\\Description\\System\\CentralProcessor\\");
	CString keyPath;
	for (int i = 0; i < dwProcessorNumber; i++) {
		TCHAR cpuName[1024] = { 0 };
		DWORD nameLength = 1024;
		DWORD data_type = 0;
		keyPath.Format(_T("%s%d"), basicPath, i);
		HKEY hKey;
		LSTATUS r = RegOpenKeyEx(HKEY_LOCAL_MACHINE, keyPath, 0, KEY_QUERY_VALUE, &hKey);
		r = RegQueryValueEx(hKey, _T("ProcessorNameString"), 0, &data_type, (LPBYTE)cpuName, &nameLength);
		CString str = _T("");
		str.Format(_T("Processor %d : %s\r\n"), i, cpuName);
		pHWList->AddTail(str);
		RegCloseKey(hKey);
	}

	///////////////////////////////////////////////////////////////////////////
	//GPU 정보 얻기

	DISPLAY_DEVICE dd = { 0 };
	dd.cb = sizeof(dd);
	CString gpuName = _T("");
	if (0 != EnumDisplayDevices(NULL, 0, &dd, 0)) {
		gpuName.Format(_T("GPU : %s\r\n"), dd.DeviceString);
		pHWList->AddTail(gpuName);
	}




	return TRUE;
}

BOOL CResourceStatus::ScanSW(CStringList *pSWList) {
	CString keyPath = _T("Software\\Microsoft\\Windows\\CurrentVersion\\Uninstall");
	HKEY hKey = NULL, hSubKey = NULL;
	DWORD dwResult = ::RegOpenKeyEx(HKEY_LOCAL_MACHINE, keyPath, 0, KEY_READ, &hKey);
	if (dwResult != ERROR_SUCCESS) {
		return FALSE;
	}
	TCHAR szKeyName[MAX_PATH] = { 0 };
	TCHAR szSWName[_MAX_FNAME] = { 0 };
	DWORD dwIndex = 0;
	DWORD dwBufferSize = MAX_PATH;
	DWORD dwLength = _MAX_FNAME;
	CString subKeyPath = _T("");
	while (ERROR_NO_MORE_ITEMS != RegEnumKeyEx(hKey, dwIndex
		, szKeyName, &dwBufferSize, NULL, NULL, NULL, NULL)) {
		subKeyPath = keyPath;
		subKeyPath += _T("\\");
		subKeyPath += szKeyName;
		dwResult = RegOpenKeyEx(HKEY_LOCAL_MACHINE, subKeyPath, 0, KEY_READ, &hSubKey);
		if (dwResult != ERROR_SUCCESS) {
			CString err = _T("");
			err.Format(_T("RegOpenKeyEx() err : %d"), GetLastError());
			AfxMessageBox(err);
			return FALSE;
		}

		if (ERROR_SUCCESS == ::RegQueryValueEx(hSubKey
			, _T("DisplayName"), NULL, NULL, (LPBYTE)szSWName, &dwLength)) {
			CString str;
			str.Format(_T("%s \r\n"), szSWName);
			pSWList->AddTail(str);
		}

		dwIndex++;
		::ZeroMemory(szSWName, sizeof(szSWName));
		::ZeroMemory(szKeyName, sizeof(szKeyName));
		dwBufferSize = MAX_PATH;
		dwLength = _MAX_FNAME;
	}

	::RegCloseKey(hKey);
	return TRUE;
}
```
