```dart
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('notification settings', () {
    testWidgets('fullsceen', (tester) async {
      final locationId = await setupTestEnvironment(tester);

      // Open the notification settings for all notifications category and change the display mode to fullscreen
      await NotificationProfileDetailsTestHelper.openTheNotificationSettings(
          tester, NotificationCategoryType.allnotifications);
      await NotificationAlertSettingsViewTestHelper.changeDisplayMode(tester, NotificationDisplayMode.fullscreen);

      // Post a patient alarm notification, generated alarm will have a unique title
      final patientAlarm = await PostNotificationHelper.postPatientAlarm(
        tester,
        LocationMatchDetailsDto(locationId: locationId),
      );

      // Since the display mode is fullscreen, the notification should be displayed right away.
      final notificationDetails = find.widgetWithText(FullscreenNotificationView, patientAlarm.title);
      expect(notificationDetails, findsOneWidget,
          reason: 'The notification should be displayed in fullscreen mode with the correct title');

      await NavigationMgr.instance.goBack();
      await tester.pumpAndSettle();
      expect(find.byType(FullscreenNotificationView), findsNothing,
          reason: 'The fullscreen notification should be closed after pressing the back button');
      expect(find.byType(NotificationProfileDetailsView), findsOneWidget,
          reason: 'Should return to the last screen on back press. Which is the NotificationProfileDetailsView');

      await NotificationProfileDetailsTestHelper.openTheNotificationSettings(
          tester, NotificationCategoryType.allnotifications);
      await NotificationAlertSettingsViewTestHelper.changeDisplayMode(tester, NotificationDisplayMode.banner);

      await PostNotificationHelper.postPatientAlarm(
        tester,
        LocationMatchDetailsDto(locationId: locationId),
      );

      // Since the display mode is banner, the notification should not be displayed right away.
      final fullscreenView = find.byType(FullscreenNotificationView);
      expect(fullscreenView, findsNothing, reason: 'The notification should not be displayed in fullscreen mode');
    });

    testWidgets('can complete', (tester) async {
      final locationId = await setupTestEnvironment(tester);

      await NotificationProfileDetailsTestHelper.openTheNotificationSettings(
          tester, NotificationCategoryType.allnotifications);
      await NotificationAlertSettingsViewTestHelper.changeCanComplete(tester, true);

      final patientAlarm = await PostNotificationHelper.postPatientAlarm(
        tester,
        LocationMatchDetailsDto(locationId: locationId),
      );

      // Verify that the notification was received and can be completed
      await ActiveNotificationsTestHelper.open(tester);
      await ActiveNotificationsTestHelper.openActiveNotification(tester, uniqueTitle: patientAlarm.title);
      await NotificationDetailsTestHelper.acceptTheNotification(tester);
      await ActiveNotificationsTestHelper.openActiveNotification(tester, uniqueTitle: patientAlarm.title);
      await NotificationDetailsTestHelper.completeTheNotification(tester);

      // After completing the notification, it should be removed from the active notifications list
      await ActiveNotificationsTestHelper.verifyNotificationDoesNotExist(tester, uniqueTitle: patientAlarm.title);
    });
  });
}

/// Sets up the test environment by ensuring the user is logged in and the notification profile is created.
/// Returns the location ID of the room for which notifications can be sent.
Future<String> setupTestEnvironment(WidgetTester tester) async {
  await TestStateMgr.instance.ensureinitialized(tester);
  await TestStateMgr.instance.ensureLoggedIn(tester);

  // Create and setup the default notification profile for these tests if it doesn't exist
  final profileName = '!Notification Settings Test Profile';
  final result = await EnvironmentSetupHelper.ensureBasicNotificationSetup(newProfileName: profileName);
  expect(result.isSuccessful, true, reason: 'Failed to setup environment for test: ${result.error}');

  final locationId = result.body!;

  await AllNotificationProfilesTestHelper.open(tester);
  await AllNotificationProfilesTestHelper.selectProfile(tester, profileName);

  return locationId;
}
```
