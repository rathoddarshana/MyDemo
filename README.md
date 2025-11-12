# MyDemo

https://teams.microsoft.com/l/message/19:62d47693-c760-44a9-9f23-392626d30448_c03061d4-00c7-4421-b3af-cb6754b85ea0@unq.gbl.spaces/1761813668398?context=%7B%22contextType%22%3A%22chat%22%7D




/**
  **************************************************************************************************
  * @file      MOSkinCalibrationCompleteScreen.tsx
  * @memberof  Screens
  * @author    Genesys Electronics Design Team
  * @version   see index.ts in application root
  * @since     see index.ts in application root
  * @copyright 2024 Genesys Electronics Design Pty Ltd
  * @license
  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR
  IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND
  FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR
  CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
  DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
  DATA, OR PROFITS OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER
  IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT
  OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
  * @description
1. General

- MOSkin Calibration Complete Screen

****************************************************************************************************
2. Revision History - see index.ts in application root.
****************************************************************************************************
**/
import React, { useEffect, useState } from "react";
import { Fragment, useCallback, useMemo } from "@gd-react-native/react";
import { StyleSheet, View } from "@gd-react-native/react-native";
import ScreenContainer from "@components/containers/ScreenContainer";
import StandardButton from "@components/buttons/StandardButton";
import * as NAVIGATION from "@constants/navigation/navigation";
import Header1Text from "@components/text/Header1Text";
import RdInfoTable from "@components/tables/RdInfoTable";
import ElementContainer from "@components/containers/ElementContainer";
import BrandedHeader from "@components/display/BrandedHeader";
import * as SCREENS from "@constants/navigation/screens";
import { NativeStackScreenProps } from "@gd-react-native/navigation-native-stack";
import { RootStackParamList } from "@navigation/Application/MOSkinCalibrationStackNavigator";
import { useNavigationManager } from "@gd-react-native/navigation-manager";
import { useThemeManager } from "@gd-react-native/theme-manager";
import useScreenLog from "@gd-react-native/screen-log";
import { useCalibrationFlow } from "@context/calibration-flow-context";
import InfoBar from "@components/display/InfoBar";
import useAlertManager from "@gd-react-native/alert-manager";
import { useAppPeripherals } from "@context/app-peripherals-context";
import { calculateDoseDifference } from "@util/general";
import { useSlots } from "@context/slots-context";
import useAppAlert from "@hooks/app-alert";
import useCalibrationReportManager from "@hooks/calibration-report-manager";
import { ErrorType, useError } from "@gd-react-native/error";
import DataModal from "@components/modals/DataModal";
import RNFS from 'react-native-fs';
import XLSX from 'xlsx';
import DocumentPicker from "@gd-react-native/document-picker";
import useLocalFileManager, { DIRECTORY_ROOT } from "@gd-react-native/local-file-manager";
import useTimeUtility from "@gd-react-native/time-utility";
import { calculateAvailableStorage, getResumeScreen } from "@util/general";


const MODULE_ID = SCREENS.MOSKIN_CALIBRATION_NEW_COMPLETE_SCREEN;

type Props = NativeStackScreenProps<RootStackParamList,
    typeof SCREENS.MOSKIN_CALIBRATION_NEW_COMPLETE_SCREEN>;

/**
 * @component
 * @memberof    Screens
 * @description MOSkin Calibration Complete Screen
 */
const MOSkinCalibrationNewCompleteScreen: React.FC<Props> = () => {
    useScreenLog(MODULE_ID);
    const navigationManager = useNavigationManager();
    const calibrationFlow = useCalibrationFlow();
    const alertManager = useAlertManager();
    const { theme } = useThemeManager();
    const appPeripherals = useAppPeripherals();
    const slots = useSlots();
    const { errorAlert } = useAppAlert();
    const calibrationReportManager = useCalibrationReportManager();
    const error = useError();
    const [modalVisible, setModalVisible] = useState(false);
    const [isExporting, setIsExporting] = useState(false);
    const localFileManager = useLocalFileManager();
    const { getDateTimeString } = useTimeUtility();
    const EXPORT_FILE_NAME = "Data Export";
    const { delay } = useTimeUtility();

    const onCancelBtnPress = useCallback(() => {
        alertManager.basicTwoOptionAlert(
            "Warning",
            "Are you sure you want to cancel calibration session?",
            "No",
            () => { },
            "Yes",
            () => {
                try {
                    calibrationFlow.cancelCalibration();
                    navigationManager.replaceScreen(NAVIGATION.MOSKIN_DASHBOARD_SCREEN);
                } catch {
                    errorAlert(MODULE_ID, "Error cancelling calibration.");
                }
            },
        );
    }, [alertManager, calibrationFlow, errorAlert, navigationManager]);

    const onSaveCalibrationPress = useCallback(() => {
        alertManager.basicTwoOptionAlert(
            "Warning",
            "Calibration data will not be available on this app after this session ends. Please ensure you have downloaded/emailed the Calibration Report. Do you wish to continue?",
            "No",
            () => { },
            "Yes",
            async () => {
                let errorOccurred: ErrorType = null;
                try {
                    // errorOccurred = await calibrationFlow.setCalibrationParams(true);
                    // if (!errorOccurred) {
                    //     alertManager.basicAlert("Success", "Calibration parameters saved successfully!");
                    //     calibrationFlow.finishCalibration();
                    navigationManager.replaceScreen(NAVIGATION.MOSKIN_DASHBOARD_SCREEN);
                    // }
                } catch (e) {
                    //errorOccurred = error.report(MODULE_ID, "Error caught setting calibration params.", false, e);
                }
                // if (errorOccurred) {
                //     errorAlert(MODULE_ID, "Error setting calibration params.");
                // }
            },
        );
    }, [alertManager, calibrationFlow, error, errorAlert, navigationManager]);

    const rdIds = useMemo(() => appPeripherals.rdInformation.map((rdInfo) => (
        rdInfo.deviceId)), [appPeripherals.rdInformation]);

    const calibrationDoses = useMemo(() => {
        const activeSlots = slots.getActiveSlots();
        return (new Array(4).fill(0)).map((_, index) => {
            if (activeSlots.includes(index + 1)) {
                return calibrationFlow.calibrationDose;
            }
            return "-";
        });
    }, [calibrationFlow.calibrationDose, slots]);

    const calibration = useMemo(() => {
        const activeSlots = slots.getActiveSlots();
        return (new Array(4).fill(0)).map((_, index) => {
            // if (activeSlots.includes(index + 1)) {
            //     return calibrationFlow.calibrationDose;
            // }
            return "-";
        });
    }, [calibrationFlow.calibrationDose, slots]);

    const calibrationValues = useMemo(() => appPeripherals.rdInformation.map((elem) => {
        // const rdData = calibrationFlow.getRdData(elem.euid);
        // if (rdData && rdData.calibrationValue) {
        //     return (rdData.calibrationValue / 1000000).toFixed(5);
        // }
        // return "-";

        if (elem.isDoseRead) {
            if (elem.isCommitted) {
                return 'NEW'
            }
            else return 'FACTORY'
        }
        return "-";
    }), [appPeripherals.rdInformation, calibrationFlow]);

    const measuredDoses = useMemo(() => appPeripherals.rdInformation.map((elem) => {
        const rdData = calibrationFlow.getRdData(elem.euid);
        if (rdData && rdData.measuredDose) {
            return (rdData.measuredDose).toFixed(3);
        }
        return "-";
    }), [appPeripherals.rdInformation, calibrationFlow]);

    const differences = useMemo(() => {
        const newDifferences = ["-", "-", "-", "-"];
        const activeSlots = slots.getActiveSlots();
        for (let i = 0; i < activeSlots.length; i++) {
            const index = activeSlots[i] - 1;
            if (!calibrationValues[index]) {
                newDifferences[index] = "-";
            } else {
                let result = calculateDoseDifference(
                    calibrationFlow.calibrationDose,
                    Number(measuredDoses[index]),
                ).toFixed(1);
                if (Number.isNaN(Number(result))) {
                    newDifferences[index] = '-';
                }
                else {
                    newDifferences[index] = result;
                }
            }
        }
        return newDifferences;
    }, [calibrationFlow.calibrationDose, calibrationValues, measuredDoses, slots]);

    const onEmailBtnPress = useCallback(async () => {
        let errorOccurred: ErrorType = null;
        try {
            errorOccurred = await calibrationReportManager.emailReport({
                timestamp: calibrationFlow.timestamp,
                rdIds,
                calibrationDoses,
                measuredDoses,
                differences,
                newCalibrationConstants: calibrationValues,
                additionalInformation: calibrationFlow.additionalInformation,
            });
        } catch (e) {
            errorOccurred = error.report(MODULE_ID, "Error caught emailing report.", false, e);
        }

        if (errorOccurred && errorOccurred[1] === "Attachments too large.") {
            errorAlert(MODULE_ID, "Error generating and emailing report. The attachments exceed 25MB.");
        } else if (errorOccurred) {
            errorAlert(MODULE_ID, "Error generating and emailing report. Please check that the default Mail app is installed and logged in.");
        }
    }, [calibrationDoses, calibrationFlow.additionalInformation, calibrationFlow.timestamp,
        calibrationReportManager, calibrationValues, differences, error, errorAlert, measuredDoses,
        rdIds]);

    const reorderFields = (data, primaryKey, secondaryKey) => {
        return data.map(item => {
            const reordered: any = {};

            // First column → primaryKey
            reordered[primaryKey] = item[primaryKey];

            // Second column → secondaryKey
            if (secondaryKey && item[secondaryKey] !== undefined) {
                reordered[secondaryKey] = item[secondaryKey];
            }

            // Add the rest of the keys in their original order
            Object.keys(item).forEach(key => {
                if (key !== primaryKey && key !== secondaryKey) {
                    reordered[key] = item[key];
                }
            });

            return reordered;
        });
    };

    const setColumnWidths = (data, worksheet) => {
        const keys = Object.keys(data[0]);
        worksheet['!cols'] = keys.map(key => {
            const maxLength = data.reduce((acc, row) => {
                const val = row[key] ? row[key].toString() : "";
                return Math.max(acc, val.length);
            }, key.length);
            return { wch: maxLength + 2 }; // add padding
        });
    }

    const exportDataToExcel = async (jsonData, fileName = 'data.xlsx') => {
        try {
            const reorderedData = reorderFields(jsonData, 'deviceId', 'euid');

            // Convert JSON to worksheet
            const worksheet = XLSX.utils.json_to_sheet(reorderedData);
            // Auto-size columns
            setColumnWidths(reorderedData, worksheet);
            const workbook = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(workbook, worksheet, 'Sheet1');

            // Write workbook to base64
            const wbout = XLSX.write(workbook, { type: 'base64', bookType: 'xlsx' });

            // Define a safe export directory
            const exportDir = `${RNFS.DocumentDirectoryPath}/Csv`;

            // Ensure directory exists
            const dirExists = await RNFS.exists(exportDir);
            if (!dirExists) {
                await RNFS.mkdir(exportDir);
            }

            // Final file path
            const filePath = `${exportDir}/${fileName}`;
            console.warn("File=============", filePath);
            // Write file
            await RNFS.writeFile(filePath, wbout, 'base64');
            console.log('Excel file saved to:', filePath);

            // Optional: trigger next step
            await onSetExportPassword(filePath);

        } catch (error) {
            console.error('Error exporting data to Excel:', error);
            alertManager.basicAlert("Error", "Export failed. Please try again.");
        }
    };

    const onSetExportPassword = useCallback(async (sourceFilePath?: string) => {
        let errorOccurred: ErrorType = null;

        try {
            const availableStorage = await calculateAvailableStorage();
            if (availableStorage < 0) {
                error.report(MODULE_ID, "Not enough space for export.", false);
                alertManager.basicAlert("Error", "Not enough space for export.");
                return;
            }

            const [docErr, response] = await DocumentPicker.pickDirectory();
            if (docErr || response === null) {
                errorOccurred = error.report(MODULE_ID, "Error picking directory.", false, docErr);
            }

            if (response && sourceFilePath) {
                let trimmedPath = response.uri;
                if (trimmedPath.startsWith("file:///")) {
                    trimmedPath = trimmedPath.substring("file:///".length);
                }

                if (trimmedPath.startsWith(DIRECTORY_ROOT)) {
                    alertManager.basicAlert(
                        "Error",
                        "You cannot export to the app's folder location. Please choose another location and try again.",
                    );
                    return;
                }

                setIsExporting(true);

                const [err, dateTimeString] = getDateTimeString();
                if (err || !dateTimeString) {
                    errorOccurred = error.report(MODULE_ID, "Error generating timestamp.", false);
                } else {
                    const fileTimestamp = dateTimeString.replace(/:/g, ".");
                    const destFilePath = `${response.uri}/${EXPORT_FILE_NAME}_${fileTimestamp}.xlsx`;

                    const exportErr = await localFileManager.exportExcelFile(sourceFilePath, destFilePath);
                    if (exportErr) {
                        errorOccurred = error.report(MODULE_ID, "Error exporting Excel file.", false, exportErr);
                    } else {
                        alertManager.basicAlert("Success", "Excel file successfully exported");
                    }
                }
            }
        } catch (e) {
            errorOccurred = error.report(MODULE_ID, "Error exporting Excel file.", false, e);
        }

        if (errorOccurred) {
            alertManager.basicAlert("Error", "Error exporting Excel file. Please try again.");
        }
        setIsExporting(false);
    }, [alertManager, error, getDateTimeString, localFileManager]);

    const onDownloadBtnPress = useCallback(async () => {
        const repeatedData = Array(4)               // create an array of length 4
            .fill(appPeripherals.rdInformation)                       // fill each slot with sampleData
            .flat();
        await exportDataToExcel(repeatedData)
    }, []);

    useEffect(() => {
        console.log("This called.....")
    }, [])

    return (
        <>
            <View style={styles.headerContainer}>
                <Header1Text style={styles.headerText}>Calibration</Header1Text>
            </View>
            <Fragment>
                <ScreenContainer>
                    <BrandedHeader />
                    <ElementContainer>
                        <RdInfoTable
                            hideSlotNumberAndReadStatus
                            value1Header="RD ID"
                            value1Data={rdIds}
                            value2Header="Calibration"
                            value2Data={calibrationValues}
                            value3Header="Calibration Dose (cGy)"
                            value3Data={calibrationDoses}
                            value4Header="Measured Dose (cGy)"
                            value4Data={measuredDoses}
                            value5Header="Difference (%)"
                            value5Data={differences}
                        />
                    </ElementContainer>
                    <ElementContainer style={styles.buttonsContainer}>
                        <StandardButton
                            title="Email Calibration Report"
                            testID="Email Calibration Report"
                            onPress={onEmailBtnPress}
                        />
                        <StandardButton
                            title="Download Report"
                            testID="Cancel Calibration"
                            onPress={onDownloadBtnPress}
                        />
                        <StandardButton
                            title="Complete Calibration"
                            testID="Save New Calibration"
                            onPress={onSaveCalibrationPress}
                        />
                    </ElementContainer>
                    <DataModal
                        visible={modalVisible}
                        onClose={() => setModalVisible(false)}
                        onProceed={() => setModalVisible(false)}
                    />
                </ScreenContainer>
                <InfoBar />
            </Fragment>
        </>
    );
};

const styles = StyleSheet.create({
    completeMessageContainer: {
        width: "100%",
        padding: 35,
        alignItems: "center",
        borderRadius: 16,
    },
    buttonsContainer: {
        flexDirection: "row",
        width: "100%",
        justifyContent: "space-between",
    },
    headerContainer: {
        flexDirection: "row",
        alignItems: "center",
        justifyContent: "space-between",
        paddingHorizontal: 16,
        paddingVertical: 22,
        backgroundColor: "#fff",
        elevation: 2, // ✅ Adds shadow on Android
        shadowColor: "#000", // ✅ Adds shadow on iOS
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.1,
        shadowRadius: 3,
        borderBottomWidth: 1,
        borderColor: "#eee",
    },
    headerText: {
        fontSize: 16,
        fontWeight: "bold",
        flex: 1,
        textAlign: "center"
    },
});


export async function calculateAvailableStorage() {
    const osReserved = 20000; // Reserved for OS
    const otherAppReserved = 5000; // Reserved for other apps
    const [, storage] = await Vitals.getStorage();
    const totalAvailableStorage = Number(storage?.total) - (osReserved + otherAppReserved);
    const actualAvailableStorage = totalAvailableStorage / 4; // To allow for backup and export
    return actualAvailableStorage / 1024;
}

    const exportExcelFile = useCallback(async (
        sourceFilePath: string,   // the Excel file you generated with exportDataToExcel
        outputPath: string,       // where the user wants to save it
    ): Promise<ErrorType> => {
        try {
            // Ensure the uri is in the correct format
            const [platformErr, uri] = Platform.select<string>({
                android: outputPath,
                ios: decodeURIComponent(outputPath)?.replace?.("file://", ""),
            });

            if (platformErr || !uri) {
                return error.report(MODULE_ID, "Error reformatting uri.", false);
            }

            // Copy the Excel file to the desired location
            try {
                await FS.copyFile(sourceFilePath, uri);
                console.log("Excel file exported to:", uri);
            } catch (copyErr) {
                return error.report(MODULE_ID, "Error copying Excel file.", false, copyErr);
            }

        } catch (e) {
            return error.report(MODULE_ID, "Error occurred during Excel export.", false, e);
        }
        return null;
    }, [error]);

   const getDateTimeString = useCallback((): DataWithErrorType<string> => {
        const utcMsSinceEpoch = getLocalMsSinceEpoch();
        const [err, dateTimeString] = epochToDateTimeString(utcMsSinceEpoch / 1000);
        if (!err && dateTimeString !== null) {
            return [null, dateTimeString];
        }
        return [error.report(MODULE_ID, "Invalid Date Time String", false, err), null];
    }, [epochToDateTimeString, error, getLocalMsSinceEpoch]);

export default MOSkinCalibrationNewCompleteScreen;

