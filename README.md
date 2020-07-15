# unity-ddt

***Work in progress***

This repo is intended to host the libraries necessary to implement Data-Driven Test with UNITY C testing framework.

For now we just puke a first implementation here..

Main issue right now is that test will stop on the first failing test case. Without providing a fix for this, the component hinders some of the UNITY beauty (seeing several related tests case fail with expected and actual values actually helps *a lot*).

We put the idea in the fridge until better times come for this to evolve to overcome its limitations.

----

uint16_t DDTNumber;
const char* DDTMessage;

typedef void (*iDDTSteps)(const void* tcDescriptor);

char* DDT_Message(const char* message)
{
	static char message_[255];
	snprintf(message_, sizeof(message_), "%s [TC-%02d: %s]", message, DDTNumber, DDTMessage);
	return message_;
}

void DDT_ExecuteTestSteps(iDDTSteps testSteps, const void* tcList, uint16_t tcListSize, uint16_t tcListTypeSize)
{
	for (DDTNumber = 0; DDTNumber < tcListSize; DDTNumber++)
	{
		setUp();
		DDTMessage = "";
		(*testSteps)(&((char*)tcList)[DDTNumber * tcListTypeSize]);
		tearDown();
	}
}

#define COUNTOF(s_) sizeof(s_)/sizeof(s_[0])
#define DDT_EXECUTE_ALL(api_, list_) do { DDT_ExecuteTestSteps(api_, list_, COUNTOF(list_), sizeof(list_[0])); } while(0)
#define DDT_EXECUTE_ONE(api_, list_, tc_) do { DDT_ExecuteTestSteps(api_, &list_[tc_], 1, sizeof(list_[0])); } while(0)

typedef struct {
	const char* message;
	uint16_t logSizeWithMetadata;
	uint16_t programBlockSize;
	uint16_t eraseBlockSize;
	uint16_t slot;
	uint16_t expected;
} sGeneralTestCase;

sGeneralTestCase prvOffsetTestCases[] = {
	{	.message = "GIVEN slot with no padding, not fitting in program block, THEN offset OK",
		.logSizeWithMetadata = 128,	.programBlockSize = 64, .slot = 0, .expected = (64*2)*0+1 },

	{	.message = "GIVEN slot with padding, not fitting in program block, when prvOffset, THEN offset OK",
		.logSizeWithMetadata = 128,	.programBlockSize = 64, .slot = 0, .expected = (64*2)*0 +1},
};

void prvOffsetSteps(const void* tcDescriptor)
{
	const sGeneralTestCase *tc = (sGeneralTestCase*)tcDescriptor;
	DDTMessage = tc->message;

	// given
	givenLogSizeWithMetadata(DATASTORAGE_1, tc->logSizeWithMetadata);
	givenProgramBlockSize(DATASTORAGE_1, tc->programBlockSize);
	// when
	DataStorage_Init();
	uint32_t s0 = prvOffsetOf(DATASTORAGE_1, tc->slot);
	// then
	TEST_ASSERT_EQUAL_MESSAGE(tc->expected, s0, DDT_Message(""));
}


void test_DDT_WhenPrvOffset(void)
{
	DDT_EXECUTE_ONE(prvOffsetSteps, prvOffsetTestCases, 0);
}

void test_DDT_WhenPrvOffset1(void)
{
	DDT_EXECUTE_ONE(prvOffsetSteps, prvOffsetTestCases, 1);
}

void test_DDT_WhenPrvOffsetAll(void)
{
	DDT_EXECUTE_ALL(prvOffsetSteps, prvOffsetTestCases, 1);
}
