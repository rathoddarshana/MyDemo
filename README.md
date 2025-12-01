# MyDemo


react-native bundle
--dev false
--entry-file index.ts
--bundle-output ios/main.jsbundle
--platform ios
--assets-dest ios





npx npx react-native bundle \
  --entry-file index.ts \
  --platform ios \
  --dev false \
  --bundle-output ios/main.jsbundle \
  --assets-dest ios



export type EmailConfig = {
    subject: string,
    recipients: string[],
    body: string,
    allowLargeAttachments?: boolean,
    attachments?: string[];
};


/** @namespace  */
/**
  **************************************************************************************************
  * @file      session-report-manager.ts
  * @memberof  CalibrationReportManager
  * @description Hook for generating, exporting and emailing calibration reports
****************************************************************************************************
**/

import { useCallback, useMemo } from "@gd-react-native/react";
import * as APP_MODULES from "@constants/app-modules";
import { DataWithErrorType, ErrorType, useError } from "@gd-react-native/error";
import { useAuthenticationManager } from "@gd-react-native/authentication-manager";
import useReportManager, { EmailConfig, ReportConfig } from "@gd-react-native/report-manager";
import FS from "@gd-react-native/fs";
import RNFS from "react-native-fs";
import XLSX from "xlsx";
import { CSS } from "./session-report-manager";

export type CalibrationInfo = {
    timestamp: Date;
    rdIds: string[];
    calibrationDoses: (string | number)[];
    measuredDoses: (string | number)[];
    differences: (string | number)[];
    newCalibrationConstants: (string | number)[];
    additionalInformation: string;
};

export interface CalibrationReportManagerType {
    emailReport: (info: CalibrationInfo) => Promise<ErrorType>;
}

const MODULE_ID = APP_MODULES.CALIBRATION_REPORT_MANAGER;
const REPORT_HEIGHT = 3508;
const REPORT_WIDTH = 2480;
const LOGO_IMAGE_PATH = `${FS.MainBundlePath}/assets/app/assets/images/moskin-logo.png`;

/** ✅ Helper: Generate HTML header */
function getHeader() {
    return `
        <header>
            <div>
                <img class="logo" img-id="logo">
                <h1>Calibration Report</h1>
            </div>
        </header>
    `;
}

/** ✅ Helper: Generate HTML footer */
function getFooter(pageNum: number, totalPages: number) {
    return `
        <footer>
            <p class="page-number">
                Page ${pageNum} of ${totalPages}
            </p>
        </footer>
    `;
}

/** ✅ Helper: Export Excel and return file path */
async function exportDataToExcel(
    jsonData: Record<string, any>[],
    fileName = "CalibrationData.xlsx"
): Promise<string | null> {
    try {
        const worksheet = XLSX.utils.json_to_sheet(jsonData);
        const workbook = XLSX.utils.book_new();
        XLSX.utils.book_append_sheet(workbook, worksheet, "Sheet1");

        const wbout = XLSX.write(workbook, { type: "base64", bookType: "xlsx" });
        const exportDir = `${RNFS.DocumentDirectoryPath}/Reports`;
        const dirExists = await RNFS.exists(exportDir);
        if (!dirExists) {
            await RNFS.mkdir(exportDir);
        }

        const filePath = `${exportDir}/${fileName}`;
        await RNFS.writeFile(filePath, wbout, "base64");
        return filePath;
    } catch (error) {
        console.warn("Excel export failed:", error);
        return null;
    }
}

/** ✅ Main Hook */
const useCalibrationReportManager = (): CalibrationReportManagerType => {
    const error = useError();
    const { user } = useAuthenticationManager();
    const reportManager = useReportManager();

    /** ✅ Generate HTML for PDF report */
    const generateReportHtml = useCallback((info: CalibrationInfo): DataWithErrorType<string> => {
        const header = getHeader();

        let rdInfoRows = "";
        (new Array(4).fill(0)).forEach((_, index) => {
            rdInfoRows += `
                <tr>
                    <td>${index + 1}</td>
                    <td>${info.rdIds[index] || ""}</td>
                    <td>${info.calibrationDoses[index] || ""}</td>
                    <td>${info.measuredDoses[index] || ""}</td>
                    <td>${info.differences[index] || ""}</td>
                    <td>${info.newCalibrationConstants[index] || ""}</td>
                </tr>`;
        });

        const rdInfoTable = `
            <table>
                <tr>
                    <th>Slot Num</th>
                    <th>RD ID</th>
                    <th>Calibration Dose (cGy)</th>
                    <th>Measured Dose (cGy)</th>
                    <th>Difference (%)</th>
                    <th>New Calibration Value (mV/cGy)</th>
                </tr>
                ${rdInfoRows}
            </table>`;

        const page1Body = `
            <div class="body">
                <p>Calibration Date: ${info.timestamp.toLocaleString()}</p>
                <p>Operator Name: ${user?.firstName} ${user?.lastName}</p>
                <p>Operator Email: ${user?.email}</p>
                ${rdInfoTable}
                <p>Additional Information: ${info.additionalInformation || "None"}</p>
            </div>
        `;

        const html = `${header}${page1Body}${getFooter(1, 1)}`;
        return [null, html];
    }, [user?.email, user?.firstName, user?.lastName]);

    /** ✅ Email Report with PDF + Excel attachment */
    const emailReport = useCallback(async (info: CalibrationInfo): Promise<ErrorType> => {
        const [htmlErr, html] = generateReportHtml(info);
        if (htmlErr || !html) {
            return error.report(MODULE_ID, "Error generating html.", false);
        }

        const pdfFileName = `MOSkin - Calibration Report - ${info.timestamp.toLocaleString()}.pdf`
            .replaceAll(":", ".")
            .replaceAll("/", ".");

        const images = [{ id: "logo", uri: LOGO_IMAGE_PATH }];

        const reportConfigs: ReportConfig[] = [{
            css: CSS,
            html,
            pageWidth: REPORT_WIDTH,
            pageHeight: REPORT_HEIGHT,
            images,
            fileName: pdfFileName,
        }];

        const subject = `MOSkin Calibration Report - ${info.timestamp.toLocaleString()}`;
        const bodyContent = `Please find session report attached. Performed on ${info.timestamp.toLocaleString()}.`;
        const body = `Hello,\n\n${bodyContent}\n\nFrom: ${user?.firstName} ${user?.lastName} (${user?.email})`;

        /** ✅ Export Excel file */
        const excelFilePath = await exportDataToExcel([
            {
                "Calibration Date": info.timestamp.toLocaleString(),
                "Operator": `${user?.firstName} ${user?.lastName}`,
                "Operator Email": user?.email,
                "Additional Info": info.additionalInformation,
                ...info.rdIds.reduce((acc, id, idx) => {
                    acc[`RD-${idx + 1}`] = id;
                    return acc;
                }, {} as Record<string, any>)
            }
        ]);

        /** ✅ Add attachments to EmailConfig */
        const EMAIL_CONFIG: EmailConfig = {
            subject,
            recipients: [String(user?.email)],
            body,
            attachments: excelFilePath ? [excelFilePath] : [], // ✅ Excel file path
        };

        const err = await reportManager.emailReports(EMAIL_CONFIG, reportConfigs);
        if (err) {
            if (err[1] === "Attachments too large.") {
                return error.report(MODULE_ID, "Attachments too large.", false);
            }
            return error.report(MODULE_ID, "Error emailing reports.", false);
        }
        return null;
    }, [error, generateReportHtml, reportManager, user?.email, user?.firstName, user?.lastName]);

    return useMemo(() => ({
        emailReport,
    }), [emailReport]);
};

export default useCalibrationReportManager;
