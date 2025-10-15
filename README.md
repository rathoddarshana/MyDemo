# MyDemo

const setRdInfo = useCallback((index: number, info: RdInformationType) => {
  setRdInformation((prevState) => {
    const updatedState = [...prevState];
    const existingIndex = updatedState.findIndex(rd => rd.deviceId === info.deviceId);

    if (existingIndex !== -1) {
      updatedState[existingIndex] = info; // Update existing RD
    } else {
      updatedState.push(info); // Add new RD
    }

    return updatedState;
  });
}, []);



import React, {
  createContext,
  useCallback,
  useContext,
  useMemo,
  useState,
} from "@gd-react-native/react";
import { CalibrationParamsType } from "@models/calibrationParamsModel";
import { deepCopy } from "@gd-react-native/utilities";
import { SensitivityCalibrationType } from "@models/rdInfoModel";

// ✅ Types
export type RdInformationType = {
  euid: string;
  deviceId: string;
  modelNumber: number;
  waferData: string;
  factoryCalibrationParams: CalibrationParamsType;
  systemCalibrationParams: CalibrationParamsType;
  userCalibrationParams: CalibrationParamsType;
  numberUses: number;
  maxNumberUses: number;
  thermostableCurrent: number;
  lastVtInFactory: number;
  accumulatedVoltage: number;
  maxAccumulatedVoltage: number;
  sensitivityCalibrationType: SensitivityCalibrationType;
};

export type RdDebugInfoType = {
  euid: string;
  timestamp: Date;
  temperature: number;
  voltage: number;
  current: number;
  setCurrent: number;
  datThermoStableCurrentMode: boolean;
  timeToCompleteReading: number;
  numberRetries: number;
  initialVspan: number;
  finalVspan: number;
  batteryVoltage: number;
  voltage3V3: number;
  voltage32V: number;
  externalPower: boolean;
  stateOfCharge: number;
  sensitivity: number;
};

// ✅ Default RD data
const DEFAULT_RD_INFO_DATA: RdInformationType = {
  euid: "-",
  deviceId: "-",
  modelNumber: -1,
  waferData: "-",
  thermostableCurrent: -1,
  lastVtInFactory: -1,
  factoryCalibrationParams: { gain: -1, timestamp: new Date() },
  systemCalibrationParams: { gain: -1, timestamp: new Date() },
  userCalibrationParams: { gain: -1, timestamp: new Date() },
  numberUses: -1,
  maxNumberUses: -1,
  accumulatedVoltage: -1,
  maxAccumulatedVoltage: -1,
  sensitivityCalibrationType: "default",
};

const DEFAULT_RD_DEBUG_INFO_DATA: RdDebugInfoType = {
  euid: "-",
  timestamp: new Date(),
  temperature: 0,
  voltage: 0,
  current: 0,
  setCurrent: 0,
  datThermoStableCurrentMode: false,
  timeToCompleteReading: 0,
  numberRetries: 0,
  initialVspan: 0,
  finalVspan: 0,
  batteryVoltage: 0,
  voltage3V3: 0,
  voltage32V: 0,
  externalPower: false,
  stateOfCharge: 0,
  sensitivity: 0,
};

// ✅ Initialize with 4 default RD slots for UI display
const DEFAULT_RD_INFO = [
  deepCopy(DEFAULT_RD_INFO_DATA),
  deepCopy(DEFAULT_RD_INFO_DATA),
  deepCopy(DEFAULT_RD_INFO_DATA),
  deepCopy(DEFAULT_RD_INFO_DATA),
];

const DEFAULT_RD_DEBUG_INFO = [
  deepCopy(DEFAULT_RD_DEBUG_INFO_DATA),
  deepCopy(DEFAULT_RD_DEBUG_INFO_DATA),
  deepCopy(DEFAULT_RD_DEBUG_INFO_DATA),
  deepCopy(DEFAULT_RD_DEBUG_INFO_DATA),
];

interface AppPeripheralsContextType {
  rdInformation: RdInformationType[];
  rdDebugInformation: RdDebugInfoType[];
  datTemperature: number;
  setRdInfo: (index: number, info: RdInformationType) => void;
  setRdDebugInfo: (index: number, info: RdDebugInfoType) => void;
  setDatTemperature: (temperature: number) => void;
  resetRdInfo: (index: number) => void;
  reset: () => void;
}

const AppPeripheralsContext = createContext<AppPeripheralsContextType | null>(null);

export const AppPeripheralsContextProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [rdInformation, setRdInformation] = useState<RdInformationType[]>(DEFAULT_RD_INFO);
  const [rdDebugInformation, setRdDebugInformation] = useState<RdDebugInfoType[]>(DEFAULT_RD_DEBUG_INFO);
  const [datTemperature, setDatTemperature] = useState(0);

  // ✅ Updated setRdInfo to store all read RD data in rdInformation
  const setRdInfo = useCallback((index: number, info: RdInformationType) => {
    setRdInformation((prevState) => {
      const updatedState = [...prevState];
      const existingIndex = updatedState.findIndex(rd => rd.deviceId === info.deviceId);

      if (existingIndex !== -1) {
        updatedState[existingIndex] = info; // Update existing RD
      } else if (index < updatedState.length) {
        updatedState[index] = info; // Replace default slot
      } else {
        updatedState.push(info); // Add new RD beyond initial 4
      }

      return updatedState;
    });
  }, []);

  const setRdDebugInfo = useCallback((index: number, info: RdDebugInfoType) => {
    setRdDebugInformation((prevState) => {
      const newState = [...prevState];
      newState[index] = info;
      return newState;
    });
  }, []);

  const resetRdInfo = useCallback((index: number) => {
    setRdInfo(index, deepCopy(DEFAULT_RD_INFO_DATA));
    setRdDebugInfo(index, deepCopy(DEFAULT_RD_DEBUG_INFO_DATA));
  }, [setRdDebugInfo, setRdInfo]);

  const reset = useCallback(() => {
    setRdInformation(deepCopy(DEFAULT_RD_INFO));
    setRdDebugInformation(deepCopy(DEFAULT_RD_DEBUG_INFO));
  }, []);

  const value = useMemo(() => ({
    rdInformation,
    setRdInfo,
    rdDebugInformation,
    setRdDebugInfo,
    datTemperature,
    setDatTemperature,
    resetRdInfo,
    reset,
  }), [rdInformation, setRdInfo, rdDebugInformation, setRdDebugInfo, datTemperature, setDatTemperature, resetRdInfo, reset]);

  return (
    <AppPeripheralsContext.Provider value={value}>
      {children}
    </AppPeripheralsContext.Provider>
  );
};

export const useAppPeripherals = (): AppPeripheralsContextType => (
  useContext(AppPeripheralsContext) as AppPeripheralsContextType
);
