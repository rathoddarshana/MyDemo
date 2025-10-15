# MyDemo

const connectedRdData = useMemo(() => {
  return appPeripherals.rdInformation
    .filter(rd => rd.deviceId !== "-"); // Only RDs that are currently connected
}, [appPeripherals.rdInformation]);


<RdInfoTable
  status={calibrationFlow.rdStatus}
  value1Header="RD ID"
  value1Data={connectedRdData.map(rd => rd.deviceId)}
  value2Header="Model Number"
  value2Data={connectedRdData.map(rd => rd.modelNumber.toString())}
  value3Header="Wafer Data"
  value3Data={connectedRdData.map(rd => rd.waferData)}
  value4Header="Number of Uses"
  value4Data={connectedRdData.map(rd => rd.numberUses.toString())}
  value5Header="Pre-Read Status"
  value5Data={connectedRdData.map(() => "Complete")}
  // Add more fields if needed


useEffect(() => {
  console.log("✅ Connected RDs:", connectedRdData);
  console.log("✅ All Stored RDs:", appPeripherals.rdInformation);
}, [connectedRdData, appPeripherals.rdInformation]);
