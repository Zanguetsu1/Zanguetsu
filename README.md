import robocode.*;

// Classe do seu robô
public class MeuRobo extends AdvancedRobot {
    private boolean movingForward;

    // Método principal do robô
    public void run() {
        // Definir as cores do robô
        setColors(java.awt.Color.blue, java.awt.Color.red, java.awt.Color.green);

        // Definir o movimento inicial
        setAhead(40000);
        movingForward = true;

        // Loop infinito para o comportamento do robô
        while (true) {
            // Girar o radar para procurar alvos
            setTurnRadarRight(360);

            // Mudar a direção do movimento se batermos em uma parede
            if (getX() <= 50 || getX() >= getBattleFieldWidth() - 50 ||
                    getY() <= 50 || getY() >= getBattleFieldHeight() - 50) {
                setBack(100);
                movingForward = !movingForward;
                execute();
            }
            // Executar o movimento
            if (movingForward) {
                setAhead(100);
            } else {
                setBack(100);
            }
            execute();
        }
    }

    // Método chamado quando o robô detecta um inimigo
    public void onScannedRobot(ScannedRobotEvent e) {
        // Calcular a posição do inimigo
        double absoluteBearing = getHeadingRadians() + e.getBearingRadians();
        double enemyX = getX() + e.getDistance() * Math.sin(absoluteBearing);
        double enemyY = getY() + e.getDistance() * Math.cos(absoluteBearing);

        // Girar a arma em direção ao inimigo
        double gunTurn = absoluteBearing - getGunHeadingRadians();
        setTurnGunRightRadians(gunTurn);

        // Prever a posição futura do inimigo e mirar
        double enemyHeading = e.getHeadingRadians();
        double enemyVelocity = e.getVelocity();
        double deltaTime = 0;
        double battleFieldHeight = getBattleFieldHeight(), battleFieldWidth = getBattleFieldWidth();
        double predictedX = enemyX, predictedY = enemyY;
        while ((++deltaTime) * (20.0 - 3.0 * Math.min(3.0, Math.abs(enemyVelocity))) < 
               Math.hypot(getX() - predictedX, getY() - predictedY)) {
            predictedX += Math.sin(enemyHeading) * enemyVelocity;
            predictedY += Math.cos(enemyHeading) * enemyVelocity;
            if (predictedX < 18.0 || predictedY < 18.0 || predictedX > battleFieldWidth - 18.0 ||
                    predictedY > battleFieldHeight - 18.0) {
                predictedX = Math.min(Math.max(18.0, predictedX), battleFieldWidth - 18.0);	
                predictedY = Math.min(Math.max(18.0, predictedY), battleFieldHeight - 18.0);
                break;
            }
        }
        double turnAngle = robocode.util.Utils.normalRelativeAngle(
                Math.atan2(predictedX - getX(), predictedY - getY()) - getGunHeadingRadians());
        setTurnGunRightRadians(turnAngle);
        
        // Atirar com força total
        setFire(3);
    }
}
