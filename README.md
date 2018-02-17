# WaveBoy
### [Global Game Jam Website: https://globalgamejam.org/2017/games/wave-boy/]

created by [Nick H](https://github.com/smugclimber), [Sam Levine](https://github.com/sjklevine), [Scott Tongue](https://github.com/Stongue), [Kurt Lorey](https://github.com/KurtLorey), Victoria Joh, and Ron Passaro

#### key libraries and tech <br>
Unity3D <br>
Oculus SDK <br>
VirZoom SDK <br>
Audacity <br>
Blender <br>
C# <br>

#### about this project
WaveBoy was a contest entry into the 2017 Global Game Jam, specifically at the NYC site(The largest GGJ site in the United States). The game is a VR experience played with an exercise bike and touch controls! Take on the role of Wave Boy, a heroic neighborhood paperboy who delivers both friendly waves and fresh copies of the Wave Street Journal to the citizens to Wave Town. Developed with Unity3D, SDKs, and some 3D modeling, feel free to clone and play this on your Oculus at home. As a GGJ Demo, there is only one level unfortunately, but it is a fine time! Enjoy!

### code
In order to allow users to actually 'Wave' at characters in the game and get even more points in addition to throwing and delivering their newspapers, a Ray caster system had to be implemented [rayCasterWave](#rayCasterWave) in the application's Scripts. This occurs only if the trigger on the Left, Right, or both hand controller triggers are pulled.

#### rayCasterWave
code below is quoted from: WaveBoyWaveDetector.cs
```
private void DoWaveFromDirectionVector(Ray originalFamousRay)
{
    // We actually want to do like a ton MORE rays in a box around this one,
    // and wave at whoever they all hit.  Raycasts are cheap!
    int numRaycastsX = 10;
    int numRaycastsY = 10;
    float rayBoxWidth = 5f;
    float rayBoxHeight = 5f;
    bool triggeredWave = false;
    for (int i = 0; i < numRaycastsX; i++)
    {
        for (int j = 0; j < numRaycastsY; j++)
        {
            // Make it ray
            Vector3 localDirectionAdjust = new Vector3(-rayBoxWidth / 2f + rayBoxWidth * i / (numRaycastsX - 1),
                                                       -rayBoxHeight / 2f + rayBoxHeight * j / (numRaycastsY - 1),
                                                       0);
            //Vector3 newOrigin = originalFamousRay.origin + this.transform.TransformDirection(localDirectionAdjust);
            Vector3 newOrigin = originalFamousRay.origin + localDirectionAdjust;
            Ray newRay = new Ray(newOrigin, originalFamousRay.direction);
            RaycastHit hit;
            float rayDist = 100f;

            if (!triggeredWave && Physics.Raycast(newRay, out hit, rayDist))
            {
                // Did we hit somebody?  If so, it counts as a wave, but only if they're not already waving.
                if (hit.collider.gameObject.tag == "Person")
                {
                    Debug.DrawLine(newRay.origin, hit.point, Color.magenta, 4.0f);

                    //Are they waving?
                    Animator personAnim = hit.collider.gameObject.GetComponent<Animator>();
                    int upperBodyLayer = personAnim.GetLayerIndex("UpperBody");
                    AnimatorStateInfo currentStateInfo = personAnim.GetCurrentAnimatorStateInfo(upperBodyLayer);
                    bool isWaving = currentStateInfo.IsName("UpperBody.OnWave");
                    if (!isWaving && !personAnim.GetBool("HasWaved")) {
                        // MAKE 'EM WAVE
                        personAnim.SetTrigger("OnWave");

                        // Pop a nice score object!
                        GameObject scoreObj = (GameObject)GameObject.Instantiate(scorePrefab, hit.point, Camera.main.transform.rotation);
                        ScoreShownOnHit scoreScript = scoreObj.GetComponent<ScoreShownOnHit>();
                        scoreScript.UpdateTextAndGo("+1000");

                        // TODO: Tell the gamemanager!
                        GameManager.instance.score += 1000;
                        //TODO: check if male or female
                        voice.GetComponent<PeopleSpeach>().PlayVoice();
                        //allowedToWave = false;
                        //StartCoroutine(Wait());
                        triggeredWave = true;
                        personAnim.SetBool("HasWaved", true);
                    }
                }
            }
            else
            {
                // No hit, but pretty editor line
                Debug.DrawLine(newRay.origin, newRay.origin + newRay.direction * rayDist, Color.white, 4.0f);
            }

        }
    }
}
```
