#include <opencv2/opencv.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <opencv2/imgproc/imgproc.hpp>
#include <mediapipe/framework/mediapipe.h>
#include <windows.h> // Required for cursor control on Windows

enum Gesture { NONE, HAND_OPEN, HAND_CLOSED, THUMBS_UP, PEACE_SIGN };
Gesture currentGesture = NONE;

void DetectGesture(const mediapipe::Hands::LandmarkList& landmarks) {
    // Implement a simple gesture recognition logic based on hand landmarks.
    // You can customize this logic for more gestures.
    if (landmarks.landmark_size() >= 21) {
        const auto& thumbTip = landmarks.landmark(4);
        const auto& indexFingerTip = landmarks.landmark(8);
        const auto& middleFingerTip = landmarks.landmark(12);
        const auto& pinkyFingerTip = landmarks.landmark(20);

        double thumbIndexDistance = cv::norm(cv::Point2f(thumbTip.x(), thumbTip.y()) - cv::Point2f(indexFingerTip.x(), indexFingerTip.y()));
        double thumbMiddleDistance = cv::norm(cv::Point2f(thumbTip.x(), thumbTip.y()) - cv::Point2f(middleFingerTip.x(), middleFingerTip.y()));
        double indexPinkyDistance = cv::norm(cv::Point2f(indexFingerTip.x(), indexFingerTip.y()) - cv::Point2f(pinkyFingerTip.x(), pinkyFingerTip.y()));

        if (thumbIndexDistance < 0.02 && thumbMiddleDistance < 0.02 && indexPinkyDistance > 0.1) {
            currentGesture = THUMBS_UP;
        }
        else if (thumbIndexDistance > 0.1) {
            currentGesture = HAND_OPEN;
        }
        else if (thumbIndexDistance < 0.05 && thumbMiddleDistance < 0.05 && indexPinkyDistance < 0.05) {
            currentGesture = HAND_CLOSED;
        }
        else {
            currentGesture = NONE;
        }
    }
    else {
        currentGesture = NONE;
    }
}

int main() {
    cv::VideoCapture cap(0); // Open the default camera (usually your webcam)
    if (!cap.isOpened()) {
        std::cerr << "Error: Couldn't access the camera." << std::endl;
        return -1;
    }

    cv::Mat frame;
    cv::namedWindow("AR Hand Mouse", cv::WINDOW_NORMAL);
    cv::setWindowProperty("AR Hand Mouse", cv::WND_PROP_FULLSCREEN, cv::WINDOW_FULLSCREEN);

    // Initialize MediaPipe's Hand Tracking module
    mediapipe::Hands hands;

    bool cursorControlEnabled = true;
    POINT previousCursorPos;
    double cursorSpeed = 3.0;

    while (true) {
        cap >> frame; // Capture a frame from the camera
        if (frame.empty()) {
            std::cerr << "Error: Frame is empty." << std::endl;
            break;
        }

        // Perform hand tracking using MediaPipe
        cv::cvtColor(frame, frame, cv::COLOR_BGR2RGB);
        mediapipe::Hands::Result result = hands.Process(frame);

        if (result.has_hands() && !result.hands().empty()) {
            for (const auto& hand : result.hands()) {
                const auto& landmarks = hand.landmarks();
                DetectGesture(landmarks);

                // Display hand landmarks
                for (const auto& landmark : landmarks.landmark()) {
                    cv::Point landmarkPt(static_cast<int>(landmark.x() * frame.cols), static_cast<int>(landmark.y() * frame.rows));
                    cv::circle(frame, landmarkPt, 5, cv::Scalar(0, 0, 255), -1);
                }

                if (cursorControlEnabled) {
                    // Implement AI control logic to move the mouse cursor here
                    if (currentGesture == HAND_OPEN) {
                        // Move the cursor based on hand position
                        const auto& indexFinger = hand.landmarks(8);
                        int newX = static_cast<int>(indexFinger.x() * GetSystemMetrics(SM_CXSCREEN));
                        int newY = static_cast<int>((1.0 - indexFinger.y()) * GetSystemMetrics(SM_CYSCREEN));

                        // Calculate cursor movement based on the previous position and cursor speed
                        int dx = static_cast<int>((newX - previousCursorPos.x) / cursorSpeed);
                        int dy = static_cast<int>((newY - previousCursorPos.y) / cursorSpeed);

                        // Move the cursor
                        SetCursorPos(previousCursorPos.x + dx, previousCursorPos.y + dy);
                    }
                    else if (currentGesture == HAND_CLOSED) {
                        // Simulate a left-click when the hand is closed
                        mouse_event(MOUSEEVENTF_LEFTDOWN, 0, 0, 0, 0);
                        mouse_event(MOUSEEVENTF_LEFTUP, 0, 0, 0, 0);
                    }
                }

                // Save the current cursor position for reference
                previousCursorPos.x = static_cast<int>(indexFinger.x() * GetSystemMetrics(SM_CXSCREEN));
                previousCursorPos.y = static_cast<int>((1.0 - indexFinger.y()) * GetSystemMetrics(SM_CYSCREEN));
            }
        }

        // Display the current gesture on the screen
        std::string gestureText;
        if (currentGesture == HAND_OPEN) {
            gestureText = "Hand Open";
        }
        else if (currentGesture == HAND_CLOSED) {
            gestureText = "Hand Closed";
        }
        else if (currentGesture == THUMBS_UP) {
            gestureText = "Thumbs Up";
        }

        cv::putText(frame, gestureText, cv::Point(20, 40), cv::FONT_HERSHEY_SIMPLEX, 1.5, cv::Scalar(0, 0, 255), 2);

        // Display the frame
        cv::cvtColor(frame, frame, cv::COLOR_RGB2BGR);
        cv::imshow("AR Hand Mouse", frame);

        // Check for keyboard input to toggle cursor control mode and adjust cursor speed
        char key = cv::waitKey(10);
        if (key == 'q') {
            break;
        }
        else if (key == 'c') {
            cursorControlEnabled = !cursorControlEnabled;
        }
        else if (key == '+') {
            cursorSpeed += 0.5;
        }
        else if (key == '-') {
            cursorSpeed = std::max(1.0, cursorSpeed - 0.5);
        }
    }

    cap.release();
    cv::destroyAllWindows();

    return 0;
}
