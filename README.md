# MyDemo

const connectedRdData = useMemo(() => {
  return appPeripherals.rdInformation
    .filter(rd => rd.deviceId !== "-"); // Only RDs that are currently connected
}, [appPeripherals.rdInformation]);
